---
title: Archivo de ¿Cuáles son las novedades de Azure Active Directory? | Microsoft Docs
description: Las notas de la versión sobre novedades de la sección Introducción de este conjunto de contenidos incluye seis meses de actividad. Después de 6 meses, los elementos se quitan del artículo principal y se colocan en este artículo de archivo.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 4/30/2021
ms.author: ajburnle
ms.reviewer: dhanyahk
ms.custom: it-pro, seo-update-azuread-jan, has-adal-ref
ms.collection: M365-identity-device-management
ms.openlocfilehash: fd37ee68e0174f4c43ce4d310de2cdae65883617
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109733665"
---
# <a name="archive-for-whats-new-in-azure-active-directory"></a>Archivo de ¿Cuáles son las novedades de Azure Active Directory?

El artículo principal de notas de la versión [¿Cuáles son las novedades de Azure Active Directory?](whats-new.md) contiene las actualizaciones de los últimos seis meses, mientras que este artículo contiene toda la información más antigua.

Las notas de la versión ¿Cuáles son las novedades de Azure Active Directory? proporcionan información sobre:

- Versiones más recientes
- Problemas conocidos
- Corrección de errores
- Funciones obsoletas
- Planes de cambios

---

## <a name="october-2020"></a>Octubre de 2020

### <a name="azure-ad-on-premises-hybrid-agents-impacted-by-azure-tls-certificate-changes"></a>Agentes híbridos de Azure AD local afectados por los cambios de certificado TLS de Azure

**Tipo:** Plan de cambio  
**Categoría del servicio:** N/D  
**Funcionalidad del producto:** plataforma

Microsoft está actualizando los servicios de Azure para que usen los certificados TLS de un conjunto diferente de entidades de certificación (CA) raíz. El motivo de este cambio es que los certificados de CA actuales no cumplen uno de los requisitos de la base de referencia del foro CA/Browser. Este cambio afectará a los agentes híbridos de Azure AD instalados en el entorno local que tengan entornos protegidos con una lista fija de certificados raíz y se tendrán que actualizar para confiar en los nuevos emisores de certificados.

Este cambio producirá una interrupción del servicio si no realiza ninguna acción inmediatamente. Estos agentes incluyen [conectores de Application Proxy](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AppProxy) para el acceso remoto al entorno local, agentes de [autenticación de paso a través](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) que permiten a los usuarios iniciar sesión en aplicaciones con las mismas contraseñas y agentes de la [versión preliminar del aprovisionamiento en la nube](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) que ejecutan la sincronización de AD con Azure AD. 

Si tiene un entorno con reglas de firewall establecidas para permitir llamadas salientes únicamente a la descarga de una lista de revocación de certificados (CRL) específica, deberá permitir las siguientes direcciones URL de CRL y OCSP. Para obtener detalles completos sobre el cambio y las direcciones URL de CRL y OCSP a las que se permitirá el acceso, consulte [Cambios en los certificados TLS de Azure](../../security/fundamentals/tls-certificate-changes.md).

---

### <a name="provisioning-events-will-be-removed-from-audit-logs-and-published-solely-to-provisioning-logs"></a>Los eventos de aprovisionamiento se quitarán de los registros de auditoría y se publicarán únicamente en los registros de aprovisionamiento

**Tipo:** Plan de cambio  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 
La actividad del [servicio de aprovisionamiento](../app-provisioning/user-provisioning.md) de SCIM se registra en los registros de auditoría y en los registros de aprovisionamiento. Esto incluye actividades como la creación de un usuario en ServiceNow o un grupo en GSuite, o la importación de un rol de AWS. En el futuro, estos eventos solo se publicarán en los registros de aprovisionamiento. Este cambio se está implementando para evitar eventos duplicados entre los registros, así como los costos adicionales en los que incurren los clientes que consumen los registros de Log Analytics. 

Proporcionaremos una actualización cuando se complete una fecha. Este desuso no está previsto para el año natural 2020. 

> [!NOTE]
> Esto no afecta a los eventos de los registros de auditoría fuera de los eventos de sincronización emitidos por el servicio de aprovisionamiento. Los eventos como la creación de una aplicación, la directiva de acceso condicional, un usuario del directorio, etc. se seguirán emitiendo en los registros de auditoría. [Más información](../reports-monitoring/concept-provisioning-logs.md?context=azure%2factive-directory%2fapp-provisioning%2fcontext%2fapp-provisioning-context).
 

---

### <a name="azure-ad-on-premises-hybrid-agents-impacted-by-azure-transport-layer-security-tls-certificate-changes"></a>Agentes híbridos de Azure AD local afectados por los cambios de certificado de Seguridad de la capa de transporte (TLS) de Azure

**Tipo:** Plan de cambio  
**Categoría del servicio:** N/D  
**Funcionalidad del producto:** plataforma
 
Microsoft está actualizando los servicios de Azure para que usen los certificados TLS de un conjunto diferente de entidades de certificación (CA) raíz. Se realizará una actualización porque los certificados de CA actuales no cumplen uno de los requisitos de la base de referencia del foro CA/Browser. Este cambio afectará a los agentes híbridos de Azure AD instalados en el entorno local que tengan entornos protegidos con una lista fija de certificados raíz. Estos agentes deberán actualizarse para confiar en los nuevos emisores de certificados.

Este cambio producirá una interrupción del servicio si no realiza ninguna acción inmediatamente. Estos agentes incluyen: 
- [Conectores de Application Proxy](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AppProxy) para el acceso remoto al entorno local. 
- Agentes de [autenticación de paso a través](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) que permiten a los usuarios iniciar sesión en aplicaciones con las mismas contraseñas.
- Agentes de la [versión preliminar del aprovisionamiento en la nube](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) que realizan la sincronización de AD con Azure AD. 

Si tiene un entorno con reglas de firewall establecidas para permitir llamadas salientes únicamente a la descarga de una lista de revocación de certificados (CRL) específica, deberá permitir las direcciones URL de CRL y OCSP. Para obtener detalles completos sobre el cambio y las direcciones URL de CRL y OCSP a las que se permitirá el acceso, consulte [Cambios en los certificados TLS de Azure](../../security/fundamentals/tls-certificate-changes.md).
 
---

### <a name="azure-active-directory-tls-10-tls-11-and-3des-deprecation-in-us-gov-cloud"></a>Desuso de Azure Active Directory TLS 1.0, TLS 1.1 y 3DES en la nube de US Gov

**Tipo:** Plan de cambio  
**Categoría del servicio:** todas las aplicaciones de Azure AD  
**Funcionalidad del producto:** Estándares
 
Azure Active Directory dejará de usar los siguientes protocolos a partir del 31 de marzo de 2021:
- TLS 1.0
- TLS 1.1
- Conjunto de cifrado 3DES (TLS_RSA_WITH_3DES_EDE_CBC_SHA)

Todas las combinaciones de cliente-servidor y explorador-servidor deben usar los conjuntos de cifrado modernos y TLS 1.2 para mantener una conexión segura con Azure Active Directory para los servicios de Azure, Office 365 y Microsoft 365.

Los entornos afectados son:
- Azure US Gov
- [Office 365 GCC High y DoD](/microsoft-365/compliance/tls-1-2-in-office-365-gcc)

Para obtener instrucciones para quitar las dependencias de los protocolos en desuso, consulte [Habilitar la compatibilidad con TLS 1.2 en el entorno para la desuso de TLS 1.1 y 1.0 de Azure AD](/troubleshoot/azure/active-directory/enable-support-tls-environment).
 
---

### <a name="assign-applications-to-roles-on-administrative-unit-and-object-scope"></a>Asignación de aplicaciones a roles en el ámbito del objeto y la unidad administrativa

**Tipo:** Nueva característica  
**Categoría del servicio:** RBAC  
**Funcionalidad del producto:** Control de acceso
 
Esta característica permite asignar una aplicación (SPN) a un rol de administrador en el ámbito de la unidad administrativa. Para obtener más información, consulte [Asignación de roles con ámbito a una unidad administrativa](../roles/admin-units-assign-roles.md).

---

### <a name="now-you-can-disable-and-delete-guest-users-when-theyre-denied-access-to-a-resource"></a>Ahora puede deshabilitar y eliminar usuarios invitados cuando se les deniega el acceso a un recurso.

**Tipo:** Nueva característica  
**Categoría del servicio:** Revisiones de acceso  
**Funcionalidad del producto:** Identity Governance
 
Deshabilitar y eliminar es un control avanzado de las revisiones de acceso de Azure AD para ayudar a las organizaciones a administrar mejor los invitados externos en grupos y aplicaciones. Si los invitados se deniegan en una revisión de acceso, **deshabilitar y eliminar** los bloqueará automáticamente para que no inicien sesión durante 30 días. Después de 30 días, se quitarán del inquilino por completo.

Para obtener más información sobre esta característica, consulte [Deshabilitación y eliminación de identidades externas con las revisiones de acceso de Azure AD](../governance/access-reviews-external-users.md#disable-and-delete-external-identities-with-azure-ad-access-reviews).
 
---

### <a name="access-review-creators-can-add-custom-messages-in-emails-to-reviewers"></a>Los creadores de la revisión de acceso pueden agregar mensajes personalizados en los mensajes de correo electrónico para los revisores

**Tipo:** Nueva característica  
**Categoría del servicio:** Revisiones de acceso  
**Funcionalidad del producto:** Identity Governance
 
En las revisiones de acceso de Azure AD, los administradores que crean revisiones ahora pueden escribir un mensaje personalizado a los revisores. Los revisores verán el mensaje en el correo electrónico que reciban, que les pedirá que completen la revisión. Para más información sobre el uso de esta característica, consulte el paso 14 de la sección [Creación de una o varias revisiones de acceso](../governance/create-access-review.md#create-one-or-more-access-reviews).

---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---october-2020"></a>Nuevos conectores de aprovisionamiento disponibles en la galería de aplicaciones de Azure AD, octubre de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros
 
Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Apple Business Manager](../saas-apps/apple-business-manager-provision-tutorial.md)
- [Apple School Manager](../saas-apps/apple-school-manager-provision-tutorial.md)
- [Code42](../saas-apps/code42-provisioning-tutorial.md)
- [AlertMedia](../saas-apps/alertmedia-provisioning-tutorial.md)
- [OpenText Directory Services](../saas-apps/open-text-directory-services-provisioning-tutorial.md)
- [Cinode](../saas-apps/cinode-provisioning-tutorial.md)
- [Global Relay Identity Sync](../saas-apps/global-relay-identity-sync-provisioning-tutorial.md)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
 
---

### <a name="integration-assistant-for-azure-ad-b2c"></a>Asistente de integración para Azure AD B2C

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 
La experiencia del asistente de integración (versión preliminar) ahora está disponible para los registros de aplicaciones de Azure AD B2C. Esta experiencia le guiará en la configuración de la aplicación para escenarios comunes. Para obtener más información, consulte [Procedimientos recomendados y recomendaciones de la plataforma de identidad de Microsoft](../develop/identity-platform-integration-checklist.md).
 
---

### <a name="view-role-template-id-in-azure-portal-ui"></a>Visualización del id. de plantilla de rol en la interfaz de usuario de Azure Portal

**Tipo:** Nueva característica  
**Categoría del servicio:** Roles de Azure  
**Funcionalidad del producto:** Control de acceso
 

Ahora puede ver el id. de plantilla de cada rol de Azure AD en Azure Portal. En Azure AD, seleccione la **descripción** del rol seleccionado. 

Se recomienda que los clientes usen los identificadores de plantilla de rol en el script y el código de PowerShell, en lugar del nombre para mostrar. El id. de plantilla de rol se puede usar para los objetos [directoryRoles](/graph/api/resources/directoryrole) y [roleDefinition](/graph/api/resources/unifiedroledefinition?view=graph-rest-beta&preserve-view=true). Para obtener más información sobre los identificadores de plantilla de rol, consulte [Roles integrados de Azure AD](../roles/permissions-reference.md).

---

### <a name="api-connectors-for-azure-ad-b2c-sign-up-user-flows-is-now-in-public-preview"></a>Los conectores de API para los flujos de usuario de registro de Azure AD B2C ahora están en versión preliminar pública

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 

Los conectores de API ahora están disponibles para su uso con Azure Active Directory B2C. Los conectores de API permiten usar las API web para personalizar los flujos de usuario de registro e integrarlos con los sistemas de nube externos. Puede usar los conectores de API para:

- Realizar la integración con flujos de trabajo de aprobación personalizados
- Validar los datos de entrada del usuario
- Sobrescribir atributos de usuario 
- Ejecutar una lógica de negocios personalizada 

 Visite la documentación [Uso de conectores de API para personalizar y extender el registro](../../active-directory-b2c/api-connectors-overview.md) para obtener más información.

---

### <a name="state-property-for-connected-organizations-in-entitlement-management"></a>Propiedad de estado para organizaciones conectadas en administración de derechos

**Tipo:** Nueva característica  
**Categoría del servicio:** **Funcionalidad del producto:** Administración de directorios Administración de derechos
 

 Todas las organizaciones conectadas tendrán ahora una propiedad adicional denominada "State". El estado controlará el uso de la organización conectada en las directivas que hacen referencia a "todas las organizaciones conectadas configuradas". El valor será "configured" (lo que significa que la organización se encuentra en el ámbito de las directivas que usan la cláusula "all") o "proposed" (lo que significa que la organización no está en el ámbito).  

Las organizaciones conectadas creadas manualmente tendrán un valor predeterminado de "configured". Mientras tanto, las creadas automáticamente (a través de directivas que permiten a cualquier usuario de Internet solicitar acceso) tendrán como valor predeterminado "proposed".  Las organizaciones conectadas creadas antes del 9 de septiembre de 2020 se establecerán en "configured". Los administradores pueden actualizar esta propiedad según sea necesario. [Más información](../governance/entitlement-management-organization.md#managing-a-connected-organization-programmatically).
 

---

### <a name="azure-active-directory-external-identities-now-has-premium-advanced-security-settings-for-b2c"></a>Las identidades externas de Azure Active Directory ahora tienen una configuración de seguridad avanzada prémium para B2C

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 
Las características de detección de riesgo y acceso condicional basado en riesgos de Identity Protection ahora están disponibles en [Azure AD B2C](../..//active-directory-b2c/conditional-access-identity-protection-overview.md). Con estas características de seguridad avanzadas, los clientes ahora pueden:
- Aprovechar Intelligent Insights para evaluar el riesgo con las aplicaciones B2C y las cuentas de usuario final. Las detecciones incluyen viajes atípicos, direcciones IP anónimas, direcciones IP vinculadas a malware e Inteligencia sobre amenazas de Azure AD. También están disponibles los informes basados en el portal y la API.
- Solucione los riesgos automáticamente mediante la configuración de directivas de autenticación adaptables para los usuarios de B2C. Los desarrolladores y administradores de aplicaciones pueden mitigar el riesgo en tiempo real. Para ello, deben exigir la autenticación multifactor (MFA) o bloquear el acceso en función del nivel de riesgo del usuario detectado, con controles adicionales disponibles en función de la ubicación, el grupo y la aplicación.
- Realice la integración con los flujos de usuario de Azure AD B2C y las directivas personalizadas. Las condiciones se pueden desencadenar desde los flujos de usuario integrados en Azure AD B2C o se pueden incorporar a las directivas personalizadas de B2C. Como sucede con otros aspectos del flujo de usuario de B2C, la mensajería de experiencia del usuario final se puede personalizar. La personalización depende de las alternativas de voz, marca y mitigación de la organización.
 
---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---october-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, octubre de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En octubre de 2020 se han agregado las siguientes 27 aplicaciones nuevas a la galería de aplicaciones con compatibilidad con la federación:

[Sentry](../saas-apps/sentry-tutorial.md), [Bumblebee - Productivity Superapp](https://app.yellowmessenger.com/user/login), [ABBYY FlexiCapture Cloud](../saas-apps/abbyy-flexicapture-cloud-tutorial.md), [EAComposer](../saas-apps/eacomposer-tutorial.md), [Genesys Cloud Integration for Azure](https://apps.mypurecloud.com/msteams-integration/), [Zone Technologies Portal](https://portail.zonetechnologie.com/signin), [Beautiful.ai](../saas-apps/beautiful.ai-tutorial.md), [Datawiza Access Broker](https://console.datawiza.com/), [ZOKRI](https://app.zokri.com/), [CheckProof](../saas-apps/checkproof-tutorial.md), [Ecochallenge.org](https://events.ecochallenge.org/users/login), [atSpoke](http://atspoke.com/login), [Appointment Reminder](https://app.appointmentreminder.co.nz/account/login), [Cloud.Market](https://cloud.market/), [TravelPerk](../saas-apps/travelperk-tutorial.md), [Greetly](https://app.greetly.com/), [OrgVitality SSO](../saas-apps/orgvitality-sso-tutorial.md), [Web Cargo Air](../saas-apps/web-cargo-air-tutorial.md), [Loop Flow CRM](../saas-apps/loop-flow-crm-tutorial.md), [Starmind](../saas-apps/starmind-tutorial.md), [Workstem](https://hrm.workstem.com/login), [Retail Zipline](../saas-apps/retail-zipline-tutorial.md), [Hoxhunt](../saas-apps/hoxhunt-tutorial.md), [MEVISIO](../saas-apps/mevisio-tutorial.md), [Samsara](../saas-apps/samsara-tutorial.md), [Nimbus](../saas-apps/nimbus-tutorial.md) y [Pulse Secure virtual Traffic Manager](../saas-apps/pulse-secure-virtual-traffic-manager-tutorial.md)

La documentación de todas las aplicaciones está disponible en https://aka.ms/AppsTutorial.

Para mostrar su aplicación en la galería de aplicaciones de Azure AD, lea los detalles aquí https://aka.ms/AzureADAppRequest.

---

### <a name="provisioning-logs-can-now-be-streamed-to-log-analytics"></a>Los registros de aprovisionamiento ahora se pueden transmitir a Log Analytics

**Tipo:** Nueva característica  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 

Publique los registros de aprovisionamiento en Log Analytics con el fin de:
- Almacenar registros de aprovisionamiento durante más de 30 días.
- Definir alertas y notificaciones personalizadas.
- Crear paneles para visualizar los registros.
- Ejecutar consultas complejas para analizar los registros. 

Para obtener información sobre el uso de la característica, consulte [Entienda cómo se integra el aprovisionamiento con los registros de Azure Monitor](../app-provisioning/application-provisioning-log-analytics.md).
 
---

### <a name="provisioning-logs-can-now-be-viewed-by-application-owners"></a>Los propietarios de la aplicación ahora pueden ver los registros de aprovisionamiento.

**Tipo:** Característica modificada  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 
Ahora puede permitir que los propietarios de aplicaciones supervisen la actividad del servicio de aprovisionamiento y solucionen problemas sin proporcionarles un rol con privilegios ni convertir TI en un cuello de botella. [Más información](../reports-monitoring/concept-provisioning-logs.md).
 
---

### <a name="renaming-10-azure-active-directory-roles"></a>Cambio de nombre de 10 roles de Azure Active Directory

**Tipo:** Característica modificada  
**Categoría del servicio:** Roles de Azure  
**Funcionalidad del producto:** Control de acceso
 
Algunos roles integrados de Azure Active Directory (AD) tienen nombres distintos de los que aparecen en el Centro de administración de Microsoft 365, el portal de Azure AD y Microsoft Graph. Esta incoherencia puede provocar problemas en los procesos automatizados. Con esta actualización, vamos a cambiar el nombre de 10 roles para que sean coherentes. En la tabla siguiente se indican los nombres de roles nuevos:

![Tabla que muestra los nombres de rol en MS Graph API y Azure Portal, y el nuevo nombre de rol propuesto en el Centro de administración de M365, Azure Portal y la API.](media/whats-new/azure-role.png)

---

### <a name="azure-ad-b2c-support-for-auth-code-flow-for-spas-using-msal-js-2x"></a>Compatibilidad de Azure AD B2C con el flujo de código de autorización para aplicaciones de página única con MSAL JS 2.x

**Tipo:** Característica modificada  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 
MSAL.js versión 2.x ahora incluye compatibilidad con el flujo de código de autorización para aplicaciones web de página única (SPA). Azure AD B2C ahora admitirá el uso del tipo de aplicación SPA en Azure Portal y el uso del flujo de código de autorización de MSAL.js con PKCE para las aplicaciones de página única. Esto permitirá que las aplicaciones SPA que usan Azure AD B2C mantengan el inicio de sesión único con exploradores más recientes y cumplan las recomendaciones del protocolo de autenticación más recientes. Comience con el tutorial [Registro de una aplicación de página única (SPA) en Azure Active Directory B2C](../../active-directory-b2c/tutorial-register-spa.md).

---

### <a name="updates-to-remember-multi-factor-authentication-mfa-on-a-trusted-device-setting"></a>Actualizaciones para recordar Multi-Factor Authentication (MFA) en una configuración de dispositivo de confianza

**Tipo:** Característica modificada  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 

Recientemente hemos actualizado la opción [Recordar Multi-Factor Authentication (MFA)](../authentication/howto-mfa-mfasettings.md#remember-multi-factor-authentication) en una característica de dispositivo de confianza para ampliar la autenticación hasta un máximo de 365 días. Las licencias de Azure Active Directory (Azure AD) Premium también pueden usar la [directiva Acceso condicional: frecuencia de inicio de sesión](../conditional-access/howto-conditional-access-session-lifetime.md#user-sign-in-frequency), que proporciona más flexibilidad para la configuración de la reautenticación.

Para lograr una experiencia de usuario óptima, se recomienda usar la frecuencia de inicio de sesión Acceso condicional para ampliar la duración de las sesiones en dispositivos de confianza, ubicaciones o sesiones de bajo riesgo como alternativa a recordar MFA en un dispositivo de confianza. Para empezar, revise nuestras [instrucciones más recientes sobre la optimización de la experiencia de reautenticación](../authentication/concepts-azure-multi-factor-authentication-prompts-session-lifetime.md).

---

## <a name="september-2020"></a>Septiembre de 2020

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---september-2020"></a>Nuevos conectores de aprovisionamiento en la galería de aplicaciones de Azure AD, septiembre de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros
 
Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Coda](../saas-apps/coda-provisioning-tutorial.md)
- [Cofense Recipient Sync](../saas-apps/cofense-provision-tutorial.md)
- [InVision](../saas-apps/invision-provisioning-tutorial.md)
- [myday](../saas-apps/myday-provision-tutorial.md)
- [SAP Analytics Cloud](../saas-apps/sap-analytics-cloud-provisioning-tutorial.md)
- [Webroot Security Awareness](../saas-apps/webroot-security-awareness-training-provisioning-tutorial.md)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
 
---
### <a name="cloud-provisioning-public-preview-refresh"></a>Actualización de la versión preliminar pública del aprovisionamiento en la nube

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento en la nube de Azure AD **Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
La actualización de la versión preliminar pública del aprovisionamiento en la nube de Azure AD Connect presenta dos mejoras importantes desarrolladas a partir de los comentarios de los clientes: 

- Experiencia de asignación de atributos a través de Azure Portal

    Con esta característica, los administradores de TI pueden asignar atributos de usuario, grupo o contacto de AD a Azure AD con varios tipos de asignación presentes hoy en día. La asignación de atributos es una característica que se usa para normalizar los valores de los atributos que fluyen desde Active Directory hasta Azure Active Directory. Puede determinar si se debe asignar directamente el valor del atributo tal como es de AD a Azure AD o usar expresiones para transformar los valores del atributo durante el aprovisionamiento de usuarios. [Más información](../cloud-sync/how-to-attribute-mapping.md)

- Experiencia del usuario de prueba o aprovisionamiento a petición

    Una vez que haya realizado la configuración, puede que quiere llevar a cabo una prueba para ver si la transformación de usuario funciona según lo esperado antes de aplicarla a todos los usuarios del ámbito. Con el aprovisionamiento a petición, los administradores de TI pueden escribir el nombre distintivo (DN) de un usuario de AD y comprobar si se está sincronizando según lo previsto. El aprovisionamiento a petición proporciona una excelente manera de asegurarse de que las asignaciones de atributos realizadas con anterioridad siguen funcionando según lo esperado. [Más información](../cloud-sync/how-to-on-demand-provision.md)
 
---

### <a name="audited-bitlocker-recovery-in-azure-ad---public-preview"></a>Recuperación de BitLocker auditada en Azure AD: versión preliminar pública

**Tipo:** Nueva característica  
**Categoría del servicio:** Administración de acceso de dispositivos  
**Funcionalidad del producto:** Administración del ciclo de vida de dispositivos
 
Ahora, cuando los administradores de TI o los usuarios finales leen las claves de recuperación de BitLocker a las que tienen acceso, Azure Active Directory genera un registro de auditoría que captura quién ha accedido a la clave de recuperación. La misma auditoría proporciona detalles del dispositivo al que está asociada la clave de BitLocker.

Los usuarios finales pueden [acceder a sus claves de recuperación a través de Mi cuenta](../user-help/my-account-portal-devices-page.md#view-a-bitlocker-key). Los administradores de TI pueden acceder a las claves de recuperación mediante la [API de claves de recuperación de BitLocker en versión beta](/graph/api/resources/bitlockerrecoverykey?view=graph-rest-beta&preserve-view=true) o a través del portal de Azure AD. Para obtener más información, vea [Ver o copiar las claves de BitLocker en el portal de Azure AD](../devices/device-management-azure-portal.md#view-or-copy-bitlocker-keys).

---

### <a name="teams-devices-administrator-built-in-role"></a>Rol integrado de Administrador de dispositivos de Teams

**Tipo:** Nueva característica  
**Categoría del servicio:** RBAC  
**Funcionalidad del producto:** Control de acceso
 
Los usuarios con el rol de [Administrador de dispositivos de Teams](../roles/permissions-reference.md#teams-devices-administrator) pueden administrar [dispositivos certificados para Teams](https://www.microsoft.com/microsoft-365/microsoft-teams/across-devices/devices) desde el centro de administración de Teams. 

Este rol permite a los usuarios ver todos los dispositivos de un solo vistazo, con la posibilidad de buscar y filtrar dispositivos. El usuario también puede comprobar los detalles de cada dispositivo, por ejemplo, la cuenta con la que se ha iniciado sesión, la marca y el modelo del dispositivo. El usuario puede cambiar la configuración en el dispositivo y actualizar las versiones de software. Este rol no concede permisos para comprobar la actividad de Teams ni la calidad de las llamadas del dispositivo.
 
---

### <a name="advanced-query-capabilities-for-directory-objects"></a>Capacidades de consulta avanzada para objetos de directorio

**Tipo:** Nueva característica  
**Categoría del servicio:** MS Graph  
**Funcionalidad del producto:** Experiencia para el desarrollador
 
Todas las nuevas capacidades de consulta presentadas para objetos de directorio en las API de Azure AD ahora están disponibles en el punto de conexión v1.0 y están listas para producción. Los desarrolladores pueden contar, buscar, filtrar y ordenar objetos de directorio y vínculos relacionados mediante los operadores estándar de OData.

Para obtener más información, vea la documentación [aquí](https://aka.ms/BlogPostMezzoGA); también puede enviar comentarios mediante esta [breve encuesta](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR_yN8EPoGo5OpR1hgmCp1XxUMENJRkNQTk5RQkpWTE44NEk2U0RIV0VZRy4u).
 
---

### <a name="public-preview-continuous-access-evaluation-for-tenants-who-configured-conditional-access-policies"></a>Versión preliminar pública: evaluación de acceso continua para inquilinos que han configurado directivas de acceso condicional

**Tipo:** Nueva característica  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
La evaluación de acceso continua (CAE) ahora está disponible en versión preliminar pública para inquilinos de Azure AD con directivas de acceso condicional. Con CAE, los eventos de seguridad críticos y las directivas se evalúan en tiempo real. Esto incluye la deshabilitación de cuentas, el restablecimiento de contraseñas y el cambio de ubicación. Para obtener más información, vea [Evaluación continua de acceso](../conditional-access/concept-continuous-access-evaluation.md).

---

### <a name="public-preview-ask-users-requesting-an-access-package-additional-questions-to-improve-approval-decisions"></a>Versión preliminar pública: preguntas adicionales a los usuarios que solicitan un paquete de acceso para mejorar las decisiones de aprobación

**Tipo:** Nueva característica  
**Categoría del servicio:** Administración de acceso de usuarios  
**Funcionalidad del producto:** Administración de derechos
 
Los administradores ahora pueden exigir que los usuarios que solicitan un paquete de acceso respondan a preguntas adicionales más allá de la justificación comercial en el portal Mi acceso de administración de derechos de Azure AD. Las respuestas de los usuarios se muestran a los aprobadores para ayudarles a tomar una decisión de aprobación de acceso más precisa. Para obtener más información, vea [Recopilación de información del solicitante adicional para su aprobación (versión preliminar)](../governance/entitlement-management-access-package-approval-policy.md#collect-additional-requestor-information-for-approval-preview).
 
---

### <a name="public-preview-enhanced-user-management"></a>Versión preliminar pública: administración de usuarios mejorada

**Tipo:** Nueva característica  
**Categoría del servicio:** User Management  
**Funcionalidad del producto:** User Management
 

El portal de Azure AD se ha actualizado para facilitar la búsqueda de usuarios en las páginas Todos los usuarios y Usuarios eliminados. Estos son algunos de los cambios de esta versión preliminar: 
- Propiedades de usuario más visibles, incluidos el identificador de objeto, el estado de sincronización de directorios, el tipo de creación y el emisor de identidades.
- Al buscar ahora se permite la búsqueda combinada de nombres, correos electrónicos e identificadores de objeto.
- Filtrado mejorado por tipo de usuario (miembro, invitado y ninguno), estado de sincronización de directorios, tipo de creación, nombre de la empresa y nombre de dominio.
- Nuevas capacidades de organización en propiedades como el nombre, el nombre principal de usuario y la fecha de eliminación.
- Un nuevo recuento de usuarios totales que se actualiza con cualquier búsqueda o filtro.

Para obtener más información, vea [Mejoras en la administración de usuarios (versión preliminar) en Azure Active Directory](../enterprise-users/users-search-enhanced.md).

---

### <a name="new-notes-field-for-enterprise-applications"></a>Nuevo campo de notas para aplicaciones empresariales

**Tipo:** Nueva característica  
**Categoría del servicio:** **Funcionalidad del producto:** Aplicaciones empresariales SSO

Puede agregar notas de texto libre a aplicaciones empresariales. Puede agregar cualquier información relevante que le ayude a administrar aplicaciones a aplicaciones empresariales. Para más información, consulte [Inicio rápido: Configuración de las propiedades de una aplicación en el inquilino de Azure Active Directory (Azure AD)](../manage-apps/add-application-portal-configure.md). 

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---september-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, septiembre de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros

En septiembre de 2020 se han agregado las siguientes 34 aplicaciones nuevas a la galería de aplicaciones con compatibilidad con la federación:

[VMware Horizon - Unified Access Gateway](), [Pulse Secure PCS](../saas-apps/vmware-horizon-unified-access-gateway-tutorial.md), [Inventory360](../saas-apps/pulse-secure-pcs-tutorial.md), [Frontitude](https://services.enteksystems.de/sso/microsoft/signup), [BookWidgets](https://www.bookwidgets.com/sso/office365), [ZVD_Server](https://zaas.zenmutech.com/user/signin), [HashData for Business](https://hashdata.app/login.xhtml), [SecureLogin](https://securelogin.securelogin.nu/sso/azure/login), [CyberSolutions MAILBASEΣ/CMSS](../saas-apps/cybersolutions-mailbase-tutorial.md), [CyberSolutions CYBERMAILΣ](../saas-apps/cybersolutions-cybermail-tutorial.md), [LimbleCMMS](https://auth.limblecmms.com/), [Glint Inc](../saas-apps/glint-inc-tutorial.md), [zeroheight](../saas-apps/zeroheight-tutorial.md), [Gender Fitness](https://app.genderfitness.com/), [Coeo Portal](https://my.coeo.com/), [Grammarly](../saas-apps/grammarly-tutorial.md), [Fivetran](../saas-apps/fivetran-tutorial.md), [Kumolus](../saas-apps/kumolus-tutorial.md), [RSA Archer Suite](../saas-apps/rsa-archer-suite-tutorial.md), [TeamzSkill](../saas-apps/teamzskill-tutorial.md), [raumfürraum](../saas-apps/raumfurraum-tutorial.md), [Saviynt](../saas-apps/saviynt-tutorial.md), [BizMerlinHR](https://marketplace.bizmerlin.net/bmone/signup), [Mobile Locker](../saas-apps/mobile-locker-tutorial.md), [Zengine](../saas-apps/zengine-tutorial.md), [CloudCADI](https://app.cloudcadi.com/login), [Simfoni Analytics](https://simfonianalytics.com/accounts/microsoft/login/), [Priva Identity & Access Management](https://my.priva.com/), [Nitro Pro](https://www.gonitro.com/nps/product-details/downloads), [Eventfinity](../saas-apps/eventfinity-tutorial.md), [Fexa](../saas-apps/fexa-tutorial.md), [Secured Signing Enterprise Portal](https://www.securedsigning.com/aad/Auth/ExternalLogin/AdminPortal), [Secured Signing Enterprise Portal AAD Setup](https://www.securedsigning.com/aad/Auth/ExternalLogin/AdminPortal), [Wistec Online](https://wisteconline.com/auth/oidc), [Oracle PeopleSoft - Protected by F5 BIG-IP APM](../saas-apps/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial.md)

La documentación de todas las aplicaciones está disponible en https://aka.ms/AppsTutorial.

Para mostrar una aplicación en la galería de aplicaciones de Azure AD, lea los detalles de https://aka.ms/AzureADAppRequest.

---

### <a name="new-delegation-role-in-azure-ad-entitlement-management-access-package-assignment-manager"></a>Nuevo rol de delegación en la administración de derechos de Azure AD: Administrador de asignaciones de paquetes de acceso

**Tipo:** Nueva característica  
**Categoría del servicio:** Administración de acceso de usuarios  
**Funcionalidad del producto:** Administración de derechos
 
Se ha agregado un nuevo rol de Administrador de asignaciones de paquetes de acceso a la administración de derechos de Azure AD para proporcionar permisos pormenorizados para administrar asignaciones. Ahora puede delegar tareas a un usuario de este rol, que puede delegar la administración de asignaciones de un paquete de acceso a un propietario empresarial. Pero un administrador de asignaciones de paquetes de acceso no puede modificar las directivas de paquetes de acceso ni otras propiedades establecidas por los administradores. 

Con este nuevo rol, se beneficia de los privilegios mínimos necesarios para delegar la administración de asignaciones y mantener el control administrativo sobre todas las demás configuraciones de paquetes de acceso. Para obtener más información, vea [Roles de administración de derechos](../governance/entitlement-management-delegate.md#entitlement-management-roles).
 
---

### <a name="changes-to-privileged-identity-managements-onboarding-flow"></a>Cambios en el flujo de incorporación de Privileged Identity Management

**Tipo:** Característica modificada  
**Categoría del servicio:** Privileged Identity Management  
**Funcionalidad del producto:** Privileged Identity Management
 
Anteriormente, la incorporación a Privileged Identity Management (PIM) requería el consentimiento del usuario y un flujo de incorporación en la hoja de PIM que incluía la inscripción en Azure AD MFA. Con la reciente integración de la experiencia de PIM en la hoja de roles y administradores de Azure AD, se elimina dicha experiencia. Cualquier inquilino con una licencia P2 válida se incorpora automáticamente a PIM.

La incorporación a PIM no tiene ningún efecto adverso directo en el inquilino. Puede contar con los siguientes cambios:
- Opciones de asignación adicionales, como activa frente a válida con hora de inicio y finalización al realizar una asignación en la hoja de roles y administradores tanto de PIM como de Azure AD. 
- Mecanismos de ámbito adicionales, como Unidades administrativas y roles personalizados, incorporados directamente en la experiencia de asignación. 
- Si es un administrador global o un administrador de roles con privilegios, puede empezar a recibir algunos correos electrónicos adicionales, como el resumen semanal de PIM. 
- También podría ver la entidad de servicio MS-PIM en el registro de auditoría relacionado con la asignación de roles. Este cambio esperado no debe afectar al flujo de trabajo normal.

 Para obtener más información, vea [Empiece a usar Privileged Identity Management](../privileged-identity-management/pim-getting-started.md).

---

### <a name="azure-ad-entitlement-management-the-select-pane-of-access-package-resources-now-shows-by-default-the-resources-currently-in-the-selected-catalog"></a>Administración de derechos de Azure AD: el panel Seleccionar de los recursos de paquete de acceso muestra ahora de forma predeterminada los recursos contenidos actualmente en el catálogo seleccionado

**Tipo:** Característica modificada  
**Categoría del servicio:** Administración de acceso de usuarios  
**Funcionalidad del producto:** Administración de derechos
 

En el flujo de creación de paquetes de acceso, en la pestaña Roles de recursos, el comportamiento del panel Seleccionar cambia. Actualmente, el comportamiento predeterminado es mostrar todos los recursos que son propiedad del usuario y los recursos agregados al catálogo seleccionado. 

Esta experiencia se modifica para mostrar de forma predeterminada solo los recursos agregados actualmente al catálogo, de modo que los usuarios puedan elegir fácilmente los recursos del catálogo. La actualización ayuda a detectar los recursos que se van a agregar a los paquetes de acceso y reduce el riesgo de agregar de forma accidental los recursos que pertenecen al usuario y que no forman parte del catálogo. Para obtener más información, vea [Creación de un paquete de acceso en la administración de derechos de Azure AD](../governance/entitlement-management-access-package-create.md#resource-roles).
 
---

## <a name="august-2020"></a>Agosto de 2020 
 
### <a name="updates-to-azure-multi-factor-authentication-server-firewall-requirements"></a>Actualizaciones de los requisitos de firewall del Servidor Microsoft Azure Multi-Factor Authentication

**Tipo:** Plan de cambio  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
A partir del 1 de octubre de 2020, los requisitos de firewall del Servidor Azure MFA requerirán intervalos IP adicionales.

Si tiene reglas de firewall de salida en la organización, actualícelas para que los servidores MFA puedan comunicarse con todos los intervalos IP necesarios. Los intervalos IP se documentan en [Requisitos de firewall de Servidor Microsoft Azure Multi-Factor Authentication](../authentication/howto-mfaserver-deploy.md#azure-multi-factor-authentication-server-firewall-requirements).

---

### <a name="upcoming-changes-to-user-experience-in-identity-secure-score"></a>Próximos cambios en la experiencia del usuario en Puntuación segura de identidad

**Tipo:** Plan de cambio  
**Categoría del servicio:** **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

Vamos a actualizar el portal de Puntuación segura de identidad para que se corresponda con los cambios introducidos en la [nueva versión](/microsoft-365/security/mtp/microsoft-secure-score-whats-new) de Puntuación de seguridad de Microsoft. 

La versión preliminar con los cambios estará disponible a principios de septiembre. Los cambios en la versión preliminar incluyen:
- "Puntuación segura de identidad" se ha cambiado a "Puntuación de seguridad de la identidad" a fin de alinear la marca con Puntuación de seguridad de Microsoft.
- Puntos normalizados para la escala estándar que se indican en porcentajes en lugar de con puntos

En esta versión preliminar, los clientes pueden alternar entre la experiencia existente y la nueva. Esta versión preliminar pública durará hasta finales de noviembre de 2020. Después de la versión preliminar, los clientes se dirigirán automáticamente a la nueva experiencia del usuario.

---

### <a name="new-restricted-guest-access-permissions-in-azure-ad---public-preview"></a>Nuevos permisos de acceso de invitado restringido en Azure AD: versión preliminar pública

**Tipo:** Nueva característica  
**Categoría del servicio:** Control de acceso   
**Funcionalidad del producto:** User Management

Hemos actualizado los permisos del nivel de directorio para los usuarios invitados. Estos permisos permiten a los administradores requerir restricciones y controles adicionales en el acceso de los usuarios invitados externos. Los administradores ahora pueden agregar restricciones adicionales para el acceso de los invitados externos a la información de pertenencia y al perfil de los usuarios y los grupos. Con esta característica de la versión preliminar pública, los clientes pueden administrar el acceso de los usuarios externos a escala mediante la ofuscación de las pertenencias a grupos, incluida la restricción de los usuarios invitados para ver la pertenencia de los grupos en los que se encuentran.

Para más información, consulte [Restricción del acceso a usuarios invitados](../enterprise-users/users-restrict-guest-permissions.md) y [Permisos predeterminados de usuario](./users-default-permissions.md).
 
---

### <a name="general-availability-of-delta-queries-for-service-principals"></a>Disponibilidad general de consultas delta para entidades de servicio

**Tipo:** Nueva característica  
**Categoría del servicio:** MS Graph  
**Funcionalidad del producto:** Experiencia para el desarrollador
 
Microsoft Graph Delta Query ahora admite el tipo de recurso en v1.0:
- Entidad de servicio

Ahora los clientes pueden realizar un seguimiento eficaz de los cambios en esos recursos. Proporciona la mejor solución para sincronizar los cambios en esos recursos con un almacén de datos local. Para aprender a configurar estos recursos en una consulta, vea [Usar la consulta delta para realizar el seguimiento de los cambios en datos de Microsoft Graph](/graph/delta-query-overview).
 
---

### <a name="general-availability-of-delta-queries-for-oauth2permissiongrant"></a>Disponibilidad general de consultas delta para oAuth2PermissionGrant

**Tipo:** Nueva característica  
**Categoría del servicio:** MS Graph  
**Funcionalidad del producto:** Experiencia para el desarrollador

Microsoft Graph Delta Query ahora admite el tipo de recurso en v1.0:
- OAuth2PermissionGrant

Los clientes ahora pueden realizar el seguimiento de los cambios de esos recursos de manera eficaz. Proporciona la mejor solución para sincronizar los cambios en esos recursos con un almacén de datos local. Para aprender a configurar estos recursos en una consulta, vea [Usar la consulta delta para realizar el seguimiento de los cambios en datos de Microsoft Graph](/graph/delta-query-overview).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---august-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, agosto de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros

En agosto de 2020 se han agregado las siguientes 25 aplicaciones nuevas a la galería de aplicaciones con compatibilidad con la federación:

[Backup365](https://portal.backup365.io/login), [Soapbox](https://app.soapboxhq.com/create?step=auth&provider=azure-ad2-oauth2), [Alma SIS](https://almau.getalma.com/), [Enlyft Dynamics 365 Connector](http://enlyft.com/), [Serraview Space Utilization Software Solutions](../saas-apps/serraview-space-utilization-software-solutions-tutorial.md), [Uniq](https://web.uniq.app/), [Visibly](../saas-apps/visibly-tutorial.md), [Zylo](../saas-apps/zylo-tutorial.md), [Edmentum - Courseware Assessments Exact Path](https://auth.edmentum.com/elf/login), [CyberLAB](https://cyberlab.evolvesecurity.com/#/welcome), [Altamira HRM](../saas-apps/altamira-hrm-tutorial.md), [WireWheel](../saas-apps/wirewheel-tutorial.md), [Zix Compliance and Capture](https://sminstall.zixcorp.com/teams/teams.php?install_request=true&tenant_id=common), [Greenlight Enterprise Business Controls Platform](../saas-apps/greenlight-enterprise-business-controls-platform-tutorial.md), [Genetec Clearance](https://www.clearance.network/), [iSAMS](../saas-apps/isams-tutorial.md), [VeraSMART](../saas-apps/verasmart-tutorial.md), [Amiko](https://amiko.web.rivero.app/), [Twingate](https://auth.twingate.com/signup), [Funnel Leasing](https://nestiolistings.com/sso/oidc/azure/authorize/), [Scalefusion](https://scalefusion.com/users/sign_in/), [Bpanda](https://goto.bpanda.com/login), [Vivun Calendar Connect](https://app.vivun.com/dashboard/calendar/connect), [FortiGate SSL VPN](../saas-apps/fortigate-ssl-vpn-tutorial.md), [Wandera End User](https://www.wandera.com/)

La documentación de todas las aplicaciones está disponible en https://aka.ms/AppsTutorial.

Para mostrar su aplicación en la galería de aplicaciones de Azure AD, lea los detalles aquí https://aka.ms/AzureADAppRequest.

---

### <a name="resource-forests-now-available-for-azure-ad-ds"></a>Los bosques de recursos ahora están disponibles para Azure AD DS. 

**Tipo:** **Categoría del servicio:** Nueva característica Azure AD Domain Services   
**Funcionalidad del producto:** Azure AD Domain Services
 
La funcionalidad de los bosques de recursos en Azure AD Domain Services ahora está disponible con carácter general. Ahora puede habilitar la autorización sin sincronización de hash de contraseñas para usar Azure AD Domain Services, incluida la autorización con tarjetas inteligentes. Para más información consulte [Conceptos y características de los conjuntos de réplicas de Azure Active Directory Domain Services (versión preliminar)](../../active-directory-domain-services/concepts-replica-sets.md).
 
---

### <a name="regional-replica-support-for-azure-ad-ds-managed-domains-now-available"></a>Ya está disponible la compatibilidad con la réplica regional para dominios administrados de Azure AD DS.

**Tipo:** Nueva característica   
**Categoría del servicio:** Azure AD Domain Services  
**Funcionalidad del producto:** Azure AD Domain Services
 
Puede ampliar un dominio administrado para que tenga más de un conjunto de réplicas por cada inquilino de Azure AD. Los conjuntos de réplicas pueden agregarse a cualquier red virtual emparejada en cualquier región de Azure que admita Azure AD Domain Services. Contar con conjuntos de réplicas adicionales en diferentes regiones de Azure facilita la recuperación geográfica ante desastres de las aplicaciones heredadas si una región de Azure se queda sin conexión. Para más información consulte [Conceptos y características de los conjuntos de réplicas de Azure Active Directory Domain Services (versión preliminar)](../../active-directory-domain-services/concepts-replica-sets.md).

---

### <a name="general-availability-of-azure-ad-my-sign-ins"></a>Disponibilidad general de Mis inicios de sesión de Azure AD

**Tipo:** Nueva característica  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Experiencias de usuario final
 
Mis inicios de sesión de Azure AD es una característica nueva que permite a los usuarios empresariales revisar su historial de inicios de sesión para comprobar si hay alguna actividad inusual. Además, esta característica posibilita a los usuarios finales comunicar si fueron o no los causantes de actividades sospechosas. Para más información sobre el uso de esta característica, consulte [Visualización de las actividades de inicio de sesión recientes](../user-help/my-account-portal-sign-ins-page.md#confirm-unusual-activity).
 
---

### <a name="sap-successfactors-hr-driven-user-provisioning-to-azure-ad-is-now-generally-available"></a>El aprovisionamiento de usuarios controlado por SuccessFactors HR de SAP para Azure AD ya está disponible con carácter general

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
Ahora puede integrar SAP SuccessFactors como el origen de identidad de autoridad con Azure AD y automatizar el ciclo de vida de identidad de un extremo a otro mediante eventos de RR. HH. como nuevas contrataciones y conclusión de contratos para impulsar el aprovisionamiento y el desaprovisionamiento de las cuentas en Azure AD. 

Para más información sobre cómo configurar el aprovisionamiento de entrada de SAP SuccessFactors para Azure AD, consulte el tutorial [Configuración del aprovisionamiento de usuarios de SAP SuccessFactors a Active Directory](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md).
 
---

### <a name="custom-open-id-connect-ms-graph-api-support-for-azure-ad-b2c"></a>Compatibilidad con Open ID Connect personalizado de MS Graph API para Azure AD B2C

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 
Anteriormente, los proveedores personalizados de Open ID Connect solo se podían agregar o administrar mediante Azure Portal. Ahora, los clientes de Azure AD B2C pueden agregarlos y administrarlos con la versión beta de Microsoft Graph API. Para obtener información sobre cómo configurar este recurso con las API, consulte [Tipo de recurso identityProvider](/graph/api/resources/identityprovider?view=graph-rest-beta&preserve-view=true).
 
---

### <a name="assign-azure-ad-built-in-roles-to-cloud-groups"></a>Asignación de roles integrados de Azure AD a grupos en la nube

**Tipo:** Nueva característica  
**Categoría del servicio:** Roles de Azure AD  
**Funcionalidad del producto:** Control de acceso

Ahora puede asignar roles integrados de Azure AD a grupos en la nube con esta nueva característica. Por ejemplo, puede asignar el rol Administrador de SharePoint al grupo Contoso_SharePoint_Admins. También puede usar PIM para convertir el grupo en miembro del rol, en lugar de conceder acceso permanente. Para obtener información sobre cómo configurar esta característica, consulte [Uso de grupos en la nube para administrar asignaciones de roles en Azure Active Directory (versión preliminar)](../roles/groups-concept.md).
 
---

### <a name="insights-business-leader-built-in-role-now-available"></a>Ya está disponible el rol integrado Coordinador de Insights de la empresa.

**Tipo:** Nueva característica  
**Categoría del servicio:** Roles de Azure AD  
**Funcionalidad del producto:** Control de acceso
 
Los usuarios con el rol Coordinador de Insights de la empresa pueden acceder a un conjunto de paneles y conclusiones a través de la [aplicación Microsoft 365 Insights](https://www.microsoft.com/microsoft-365/partners/workplaceanalytics). Esto incluye el acceso completo a todos los paneles y la funcionalidad de exploración de datos y conclusiones presentadas. Sin embargo, los usuarios con este rol no tienen acceso a las opciones de configuración del producto, que es responsabilidad del rol Administrador de Insights. Para más información acerca de este rol, consulte [Permisos de roles de administrador en Azure Active Directory](../roles/permissions-reference.md#insights-business-leader).
 
---

### <a name="insights-administrator-built-in-role-now-available"></a>El rol integrado Administrador de Insights ya está disponible.

**Tipo:** Nueva característica  
**Categoría del servicio:** Roles de Azure AD  
**Funcionalidad del producto:** Control de acceso
 
Los usuarios con el rol Administrador de Insights pueden tener acceso al conjunto completo de funcionalidades administrativas de la [aplicación Microsoft 365 Insights](https://www.microsoft.com/microsoft-365/partners/workplaceanalytics). Un usuario con este rol puede leer información de directorios, supervisar el mantenimiento del servicio, presentar vales de soporte técnico y acceder a los aspectos de configuración del administrador de Insights. Para más información sobre este rol, consulte [Permisos de roles de administrador en Azure Active Directory](../roles/permissions-reference.md#insights-administrator).
 
--- 

### <a name="application-admin-and-cloud-application-admin-can-manage-extension-properties-of-applications"></a>El administrador de aplicaciones y el administrador de aplicaciones en la nube pueden administrar las propiedades de extensión de las aplicaciones.

**Tipo:** Característica modificada  
**Categoría del servicio:** Roles de Azure AD  
**Funcionalidad del producto:** Control de acceso
 
Anteriormente, solo el administrador global podía administrar la [propiedad de extensión](/graph/api/application-post-extensionproperty?view=graph-rest-beta&tabs=http&preserve-view=true). Ahora vamos a habilitar esta funcionalidad para el administrador de la aplicación y también para el administrador de aplicaciones en la nube.
 
---

### <a name="mim-2016-sp2-hotfix-462630-and-connectors-1113010"></a>Revisión 4.6.263.0 de MIM 2016 SP2 y conectores 1.1.1301.0

**Tipo:** Característica modificada  
**Categoría del servicio:** Microsoft Identity Manager  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad

Un [paquete acumulativo de revisiones (compilación 4.6.263.0)](https://support.microsoft.com/help/4576473/hotfix-rollup-package-build-4-6-263-0-is-available-for-microsoft-ident) está disponible para Microsoft Identity Manager (MIM) 2016 Service Pack 2 (SP2). Este paquete acumulativo contiene actualizaciones para los componentes de PAM, MIM CM y el administrador de sincronización de MIM. Además, la compilación de conectores genéricos de MIM 1.1.1301.0 incluye actualizaciones para el conector de Graph.

---

## <a name="july-2020"></a>Julio de 2020

### <a name="as-an-it-admin-i-want-to-target-client-apps-using-conditional-access"></a>Como administrador de TI, quiero dirigirme a las aplicaciones cliente mediante el acceso condicional

**Tipo:** Plan de cambio   
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Con la versión de disponibilidad general (GA) de la condición de las aplicaciones de cliente en el acceso condicional, ahora las nuevas directivas se aplicarán de manera predeterminada a todas las aplicaciones cliente. Esto incluye los clientes de autenticación heredados. Las directivas existentes permanecerán sin cambios, pero la opción de alternancia *Configurar sí/no* se quitará de las directivas existentes para ver fácilmente las aplicaciones cliente que se aplican a través de la directiva. 

Al crear una nueva directiva, asegúrese de excluir los usuarios y las cuentas de servicio que siguen usando la autenticación heredada. Si no lo hace, se bloquearán. [Más información](../conditional-access/concept-conditional-access-conditions.md).
 
---

### <a name="upcoming-scim-compliance-fixes"></a>Próximas correcciones de cumplimiento de SCIM

**Tipo:** Plan de cambio  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
El servicio de aprovisionamiento de Azure AD aprovecha el estándar SCIM para la integración con las aplicaciones. Nuestra implementación del estándar SCIM está evolucionando y esperamos realizar cambios en nuestro comportamiento en torno a la realización de las operaciones de revisión, así como al establecimiento de la propiedad "activa" en un recurso. [Más información](../app-provisioning/application-provisioning-config-problem-scim-compatibility.md).
 
---

### <a name="group-owner-setting-on-azure-admin-portal-will-be-changed"></a>Se cambiará la configuración del propietario del grupo en el portal de administración de Azure

**Tipo:** Plan de cambio  
**Categoría del servicio:** Administración de grupos  
**Funcionalidad del producto:** Colaboración

La configuración del propietario en la página de configuración de grupos general puede configurarse para restringir los privilegios de asignación de propietario a un grupo limitado de usuarios en el Panel de acceso y el portal de administración de Azure. Pronto tendremos la capacidad de asignar privilegios de propietario de grupo no solo en estos dos portales de la experiencia de usuario, sino también aplicar la directiva en el back-end para proporcionar un comportamiento coherente entre los puntos de conexión, como PowerShell y Microsoft Graph. 

Comenzaremos a deshabilitar la configuración actual de los clientes que no la estén usando y ofreceremos una opción para el ámbito de los usuarios de privilegios de propietario de grupo en los próximos meses. Para obtener instrucciones sobre la actualización de la configuración de grupo, consulte la edición de la información de grupo mediante [Azure Active Directory](./active-directory-groups-settings-azure-portal.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context).

---

### <a name="azure-active-directory-registration-service-is-ending-support-for-tls-10-and-11"></a>El servicio de registro de Azure Active Directory va a finalizar la compatibilidad con TLS 1.0 y 1.1

**Tipo:** Plan de cambio  
**Categoría del servicio:** Administración y registro de dispositivos  
**Funcionalidad del producto:** plataforma

La Seguridad de la capa de transporte (TLS) 1.2 y los servidores y clientes de actualización se comunicarán pronto con el servicio Registro de dispositivos de Azure Active Directory. La compatibilidad con TLS 1.0 y 1.1 para la comunicación con el servicio Registro de dispositivos de Azure AD se retirará:
- El 31 de agosto de 2020, en todas las nubes independientes (GCC High, DoD, etc.).
- El 30 de octubre de 2020, en todas las nubes comerciales.

[Más información](../devices/reference-device-registration-tls-1-2.md) sobre TLS 1.2 para el servicio de registro de Azure AD.

---

### <a name="windows-hello-for-business-sign-ins-visible-in-azure-ad-sign-in-logs"></a>Inicios de sesión de Windows Hello para empresas visibles en registros de inicio de sesión de Azure AD

**Tipo:** Corregido  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 
Windows Hello para empresas permite a los usuarios finales iniciar sesión en máquinas Windows con un gesto (como un PIN o datos biométricos). Los administradores de Azure AD pueden querer diferenciar los inicios de sesión de Windows Hello para empresas de otros inicios de sesión de Windows como parte del recorrido de una organización para la autenticación sin contraseña. 

Los administradores ya pueden ver si una autenticación de Windows ha usado Windows Hello para empresas mediante la comprobación de la pestaña Detalles de autenticación de un evento de inicio de sesión de Windows en la hoja Inicios de sesión de Azure AD en Azure Portal. Las autenticaciones de Windows Hello para empresas incluirán "WindowsHelloForBusiness" en el campo Método de autenticación. Para obtener más información sobre cómo interpretar los registros de inicio de sesión, consulte la [documentación de registros de inicio de sesión](../reports-monitoring/concept-sign-ins.md).
 
---

### <a name="fixes-to-group-deletion-behavior-and-performance-improvements"></a>Correcciones del comportamiento de eliminación de grupos y mejoras de rendimiento

**Tipo:** Corregido  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
Anteriormente, cuando un grupo cambiaba de "en el ámbito" a "fuera de ámbito" y un administrador hacía clic en reiniciar antes de que se completara el cambio, el objeto de grupo no se eliminaba. Ahora, el objeto de grupo de la aplicación de destino se eliminará cuando salga del ámbito (deshabilitado, eliminado, sin asignar o sin pasar el filtro de ámbito). [Más información](../app-provisioning/how-provisioning-works.md#incremental-cycles).
 
---

### <a name="public-preview-admins-can-now-add-custom-content-in-the-email-to-reviewers-when-creating-an-access-review"></a>Versión preliminar pública: los administradores ya pueden agregar contenido personalizado en el correo electrónico para los revisores al crear una revisión de acceso.

**Tipo:** Nueva característica  
**Categoría del servicio:** Revisiones de acceso  
**Funcionalidad del producto:** Identity Governance
 
Cuando se crea una nueva revisión de acceso, el revisor recibe un correo electrónico que le solicita que complete la revisión de acceso. Muchos de nuestros clientes han solicitado la posibilidad de agregar contenido personalizado al correo electrónico, como información de contacto, u otro contenido adicional de soporte técnico para guiar al revisor. 

Ya disponible en la versión preliminar pública, los administradores pueden especificar contenido personalizado en el correo electrónico que se envía a los revisores mediante la adición de contenido en la sección "Opciones avanzadas" de las revisiones de acceso de Azure AD. Para obtener instrucciones sobre cómo crear revisiones de acceso, consulte [Creación de una revisión de acceso de los grupos y las aplicaciones en las revisiones de acceso de Azure AD](../governance/create-access-review.md).
 
---

### <a name="authorization-code-flow-for-single-page-apps-available"></a>Flujo de código de autorización para aplicaciones de una sola página disponibles

**Tipo:** Nueva característica  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Experiencia para el desarrollador
 
Debido a las restricciones de cookies de terceros como Safari ITP aplicadas en el explorador, las aplicaciones de página única (SPA) tendrán que usar el flujo de código de autorización en lugar del flujo implícito para mantener el SSO. MSAL.js v2.x admite ahora el flujo de código de autorización. 

Las actualizaciones correspondientes están disponibles en Azure Portal. De este modo, podrá actualizar sus aplicaciones de página única para que sean del tipo "spa" y utilicen el flujo de código de autenticación. Consulte [Inicio de sesión de los usuarios y obtención de un token de acceso en una SPA de JavaScript mediante el flujo de código de autorización](../develop/quickstart-v2-javascript-auth-code.md) para obtener más instrucciones.
 
---

### <a name="azure-ad-application-proxy-now-supports-the-remote-desktop-services-web-client"></a>Azure AD Application Proxy ya admite el cliente web de Servicios de Escritorio remoto

**Tipo:** Nueva característica  
**Categoría del servicio:** Proxy de aplicaciones  
**Funcionalidad del producto:** Control de acceso

Azure AD Application Proxy ya admite el cliente web de Servicios de Escritorio remoto (RDS). El cliente web de RDS permite a los usuarios tener acceso a la infraestructura de Escritorio remoto a través de cualquier explorador compatible con HTLM5, como Microsoft Edge, Internet Explorer 11, Google Chrome, etc. Los usuarios pueden interactuar con escritorios o aplicaciones remotos como lo harían con un dispositivo local desde cualquier lugar. Mediante el uso de Azure AD Application Proxy, puede aumentar la seguridad de la implementación de RDS mediante la exigencia de directivas de acceso condicional y autenticación previa para todos los tipos de aplicaciones cliente enriquecidas. Para obtener instrucciones, consulte [Publicación de Escritorio Remoto con Azure AD Application Proxy](../manage-apps/application-proxy-integrate-with-remote-desktop-services.md).
 
---

### <a name="next-generation-azure-ad-b2c-user-flows-in-public-preview"></a>Flujos de usuario de última generación de Azure AD B2C en versión preliminar pública

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 
La experiencia simplificada de flujo de usuario ofrece paridad de características con características en vista previa (GB) y es donde se encuentran todas las características nuevas. Los usuarios podrán habilitar nuevas características en el mismo flujo de usuario, lo que reduce la necesidad de crear varias versiones con cada nueva versión de actualización de características. Por último, la nueva experiencia de usuario fácil de usar simplifica la selección y creación de flujos de usuario. Para probarla, [cree un flujo de usuario](../../active-directory-b2c/tutorial-create-user-flows.md). 

Para más información sobre los flujos de usuario, consulte [Versiones de flujos de usuario de Azure Active Directory B2C](../../active-directory-b2c/user-flow-versions.md).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---july-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, julio de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En julio de 2020, agregamos 55 aplicaciones nuevas a la galería de aplicaciones con compatibilidad de federación:

[Clap Your Hands](http://www.rmit.com.ar/), [Appreiz](https://microsoftteams.appreiz.com/), [Inextor Vault](https://inexto.com/inexto-suite/inextor), [Beekast](https://my.beekast.com/), [Templafy OpenID Connect](https://app.templafy.com/), [PeterConnects receptionist](https://msteams.peterconnects.com/), [AlohaCloud](https://appfusions.alohacloud.com/auth), [Control Tower](https://bpm.tnxcorp.com/sso/microsoft), [Cocoom](https://start.cocoom.com/), [COINS Construction Cloud](https://sso.coinsconstructioncloud.com/#login/), [Medxnote MT](https://task.teamsmain.medx.im/authorization), [Reflekt](https://reflekt.konsolute.com/login), [Rever](https://app.reverscore.net/access), [MyCompanyArchive](https://login.mycompanyarchive.com/), [GReminders](https://app.greminders.com/o365-oauth), [Titanfile](../saas-apps/titanfile-tutorial.md), [Wootric](../saas-apps/wootric-tutorial.md), [SolarWinds Orion](https://support.solarwinds.com/SuccessCenter/s/orion-platform?language=en_US),  [OpenText Directory Services](../saas-apps/opentext-directory-services-tutorial.md), [Datasite](../saas-apps/datasite-tutorial.md), [BlogIn](../saas-apps/blogin-tutorial.md), [IntSights](../saas-apps/intsights-tutorial.md), [kpifire](../saas-apps/kpifire-tutorial.md), [Textline](../saas-apps/textline-tutorial.md), [Cloud Academy - SSO](../saas-apps/cloud-academy-sso-tutorial.md), [Community Spark](../saas-apps/community-spark-tutorial.md), [Chatwork](../saas-apps/chatwork-tutorial.md), [CloudSign](../saas-apps/cloudsign-tutorial.md), [C3M Cloud Control](../saas-apps/c3m-cloud-control-tutorial.md), [SmartHR](https://smarthr.jp/), [NumlyEngage™](../saas-apps/numlyengage-tutorial.md), [Michigan Data Hub Single Sign-On](../saas-apps/michigan-data-hub-single-sign-on-tutorial.md), [Egress](../saas-apps/egress-tutorial.md), [SendSafely](../saas-apps/sendsafely-tutorial.md), [Eletive](https://app.eletive.com/), [Right-Hand Cybersecurity ADI](https://right-hand.ai/), [Fyde Enterprise Authentication](https://enterprise.fyde.com/), [Verme](../saas-apps/verme-tutorial.md), [Lenses.io](../saas-apps/lensesio-tutorial.md), [Momenta](../saas-apps/momenta-tutorial.md), [Uprise](https://app.uprise.co/sign-in), [Q](https://q.moduleq.com/login), [CloudCords](../saas-apps/cloudcords-tutorial.md), [TellMe Bot](https://tellme365liteweb.azurewebsites.net/), [Inspire](https://app.inspiresoftware.com/), [Maverics Identity Orchestrator SAML Connector](https://www.strata.io/identity-fabric/), [Smartschool (School Management System)](https://smartschoolz.com/login), [Zepto - Intelligent timekeeping](https://user.zepto-ai.com/signin), [Studi.ly](https://studi.ly/), [Trackplan](http://www.trackplanfm.com/), [Skedda](../saas-apps/skedda-tutorial.md), [WhosOnLocation](../saas-apps/whos-on-location-tutorial.md), [Coggle](../saas-apps/coggle-tutorial.md), [Kemp LoadMaster](https://kemptechnologies.com/cloud-load-balancer/), [BrowserStack Single Sign-on](../saas-apps/browserstack-single-sign-on-tutorial.md)

La documentación de todas las aplicaciones está disponible en https://aka.ms/AppsTutorial.

Para mostrar su aplicación en la galería de aplicaciones de Azure AD, lea los detalles aquí: https://aka.ms/AzureADAppRequest.

---

### <a name="view-role-assignments-across-all-scopes-and-ability-to-download-them-to-a-csv-file"></a>Visualización de las asignaciones de roles en todos los ámbitos y capacidad de descargarlas en un archivo CSV

**Tipo:** Característica modificada  
**Categoría del servicio:** Roles de Azure AD  
**Funcionalidad del producto:** Control de acceso
 
Ahora puede ver las asignaciones de roles en todos los ámbitos de un rol en la pestaña "Roles y administradores" en el portal de Azure AD. También puede descargar estas asignaciones de roles para cada rol en un archivo CSV. Para obtener instrucciones sobre cómo ver y agregar asignaciones de roles, consulte [Visualización y asignación de roles de administrador en Azure Active Directory](../roles/manage-roles-portal.md).
 
---

### <a name="azure-multi-factor-authentication-software-development-azure-mfa-sdk-deprecation"></a>Desuso del desarrollo de software de Azure Multi-Factor Authentication (SDK de Azure MFA)

**Tipo:** Obsoleto  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
El desarrollo de software de Azure Multi-Factor Authentication (SDK de Azure MFA) alcanzó el final del ciclo de vida el 14 de noviembre de 2018, como se anunció por primera vez en noviembre de 2017. Microsoft apagará el servicio de SDK a partir del 30 de septiembre de 2020. Se producirá un error en todas las llamadas realizadas al SDK.

Si su organización usa el SDK de Azure MFA, debe realizar la migración antes del 30 de septiembre de 2020:
- SDK de Azure MFA para MIM:  Si usa el SDK con MIM, debe migrar al Servidor Azure MFA y activar Privileged Access Management (PAM) siguiendo estas [instrucciones](/microsoft-identity-manager/working-with-mfaserver-for-mim).   
- SDK de Azure MFA para aplicaciones personalizadas: Considere la posibilidad de integrar la aplicación en Azure AD y usar el acceso condicional para aplicar MFA. Para empezar, revise esta [página](../manage-apps/plan-an-application-integration.md). 

---

## <a name="june-2020"></a>Junio de 2020 

### <a name="user-risk-condition-in-conditional-access-policy"></a>Condición de riesgo de usuario de la directiva de acceso condicional

**Tipo:** Plan de cambio  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 

La compatibilidad con el riesgo de usuario en la directiva de acceso condicional de Azure AD permite crear varias directivas basadas en el riesgo de usuario. Pueden requerirse diferentes niveles de riesgo de usuario mínimo para diferentes usuarios y aplicaciones. En función del riesgo de usuario, puede crear directivas para bloquear el acceso, requerir la autenticación multifactor, proteger el cambio de contraseña o redirigir a Microsoft Cloud App Security para aplicar la directiva de sesión, como la auditoría adicional.

La condición de riesgo de usuario requiere Azure AD Premium P2 porque usa Azure Identity Protection, que es una oferta P2. Para más información sobre el acceso condicional, vea la [documentación sobre el acceso condicional de Azure AD](../conditional-access/index.yml).

---

### <a name="saml-sso-now-supports-apps-that-require-spnamequalifier-to-be-set-when-requested"></a>SSO de SAML admite ahora aplicaciones que requieren establecer SPNameQualifier cuando se solicite

**Tipo:** Corregido  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO
 
Algunas aplicaciones de SAML requieren que se devuelva SPNameQualifier en el sujeto de la aserción cuando se solicita. Ahora Azure AD responde correctamente cuando se solicita un SPNameQualifier en la directiva de NameID de la solicitud. Esto también funciona para el inicio de sesión iniciado por el SP y se seguirá el inicio de sesión iniciado por IdP.  Para más información sobre el protocolo de SAML en Azure Active Directory, vea [Protocolo SAML de inicio de sesión único](../develop/single-sign-on-saml-protocol.md).

---

### <a name="azure-ad-b2b-collaboration-supports-inviting-msa-and-google-users-in-azure-government-tenants"></a>La colaboración B2B de Azure AD permite invitar a los usuarios de MSA y Google en inquilinos de Azure Government

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 

Los inquilinos de Azure Government que usan las características de colaboración B2B ahora pueden invitar a los usuarios que tienen una cuenta de Microsoft o Google. Para averiguar si su inquilino puede usar estas funcionalidades, siga las instrucciones que se indican en [¿Cómo puedo saber si la colaboración B2B está disponible en mi inquilino de Azure US Government?](../external-identities/current-limitations.md#how-can-i-tell-if-b2b-collaboration-is-available-in-my-azure-us-government-tenant)

 
---
 
### <a name="user-object-in-ms-graph-v1-now-includes-externaluserstate-and-externaluserstatechangeddatetime-properties"></a>El objeto de usuario de MS Graph v1 ahora incluye las propiedades externalUserState y externalUserStateChangedDateTime

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 

Las propiedades externalUserState y externalUserStateChangedDateTime se pueden usar para buscar invitados B2B que aún no han aceptado sus invitaciones, así como para crear automatización, como la eliminación de usuarios que no han aceptado sus invitaciones después de un número determinado de días. Estas propiedades están ahora disponibles en MS Graph v1. Para obtener instrucciones sobre el uso de estas propiedades, consulte [Tipo de recurso del usuario](/graph/api/resources/user).
 
---

### <a name="manage-authentication-sessions-in-azure-ad-conditional-access-is-now-generally-available"></a>La administración de sesiones de autenticación en el acceso condicional de Azure AD ya está disponible con carácter general

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Las funcionalidades de administración de sesiones de autenticación permiten configurar la frecuencia con que los usuarios necesitan proporcionar credenciales de inicio de sesión y si necesitan proporcionar credenciales después de cerrar y volver a abrir los exploradores para ofrecer más seguridad y flexibilidad en el entorno.
 
Además, la administración de sesiones de autenticación se aplicaba solo a la autenticación del primer factor en los dispositivos unidos a Azure AD, unidos a Azure AD híbrido y registrados en Azure AD. Ahora la administración de sesiones de autenticación también se aplicará a MFA. Para más información, vea [Configuración de la administración de las sesiones de autenticación con el acceso condicional](../conditional-access/howto-conditional-access-session-lifetime.md).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---june-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, junio de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En junio de 2020, hemos agregado 29 nuevas aplicaciones a la galería de aplicaciones con compatibilidad de federación:

[Shopify Plus](../saas-apps/shopify-plus-tutorial.md), [Ekarda](../saas-apps/ekarda-tutorial.md), [MailGates](../saas-apps/mailgates-tutorial.md), [BullseyeTDP](../saas-apps/bullseyetdp-tutorial.md), [Raketa](../saas-apps/raketa-tutorial.md), [Segment](../saas-apps/segment-tutorial.md), [Ai Auditor](https://www.mindbridge.ai/products/ai-auditor/), [Pobuca Connect](https://app.pobu.ca/), [Proto.io](../saas-apps/proto.io-tutorial.md), [Gatekeeper](https://www.gatekeeperhq.com/), [Hub Planner](../saas-apps/hub-planner-tutorial.md), [Ansira-Partner Go-to-Market Toolbox](https://ansira.com/technology/channel-engagement), [IBM Digital Business Automation on Cloud](../saas-apps/ibm-digital-business-automation-on-cloud-tutorial.md), [Kisi Physical Security](../saas-apps/kisi-physical-security-tutorial.md), [ViewpointOne](https://team.viewpoint.com/), [IntelligenceBank](../saas-apps/intelligencebank-tutorial.md), [pymetrics](../saas-apps/pymetrics-tutorial.md), [Zero](https://www.teamzero.com/), [InStation](https://instation.invillia.com/), [edX for Business SAML 2.0 Integration](../saas-apps/edx-for-business-saml-integration-tutorial.md), [MOOC Office 365](https://mooc.office365-training.com/en/), [SmartKargo](../saas-apps/smartkargo-tutorial.md), [PKIsigning platform](https://platform.pkisigning.nl/), [SiteIntel](../saas-apps/siteintel-tutorial.md), [Field iD](../saas-apps/field-id-tutorial.md), [Curricula SAML](../saas-apps/curricula-saml-tutorial.md), [Perforce Helix Core - Helix Authentication Service](../saas-apps/perforce-helix-core-tutorial.md), [MyCompliance Cloud](https://cloud.metacompliance.com/) y [Smallstep SSH](https://smallstep.com/sso-ssh/)  

La documentación de todas las aplicaciones está disponible aquí: https://aka.ms/AppsTutorial Para mostrar su aplicación en la galería de aplicaciones Azure AD, lea los detalles aquí: https://aka.ms/AzureADAppRequest.

---

### <a name="api-connectors-for-external-identities-self-service-sign-up-are-now-in-public-preview"></a>Los conectores de API para el registro de autoservicio de identidades externas ya está disponible en versión preliminar pública

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Los conectores de API para identidades externas permiten utilizar las API web para integrar el registro de autoservicio con sistemas en la nube externos. Esto significa que ahora puede invocar las API web como pasos específicos en un flujo de registro para desencadenar flujos de trabajo personalizados basados en la nube. Por ejemplo, puede usar conectores de API para:

- Integrarse con un flujo de trabajo de aprobación personalizado.
- Realizar correcciones de identidad
- Validar los datos de entrada del usuario
- Sobrescribir atributos de usuario
- Ejecutar una lógica de negocios personalizada

Para más información sobre todas las experiencias posibles con los conectores de API, vea [Uso de conectores de API para personalizar y extender el registro de autoservicio](../external-identities/api-connectors-overview.md) o [Personalización del registro de autoservicio de las identidades externas con las integraciones de API web](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/customize-external-identities-self-service-sign-up-with-web-api/ba-p/1257364#.XvNz2fImuQg.linkedin).
 
---

### <a name="provision-on-demand-and-get-users-into-your-apps-in-seconds"></a>Aprovisionamiento a petición e integración de usuarios en las aplicaciones en cuestión de segundos

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
El servicio de aprovisionamiento de Azure AD actualmente funciona de forma cíclica. El servicio se ejecuta cada 40 minutos. La [funcionalidad de aprovisionamiento a petición](https://aka.ms/provisionondemand) permite elegir un usuario y aprovisionarlo en segundos. Esta funcionalidad permite solucionar problemas de aprovisionamiento rápidamente, sin tener que reiniciar para forzar el inicio del ciclo de aprovisionamiento. 
 
---

### <a name="new-permission-for-using-azure-ad-entitlement-management-in-graph"></a>Nuevo permiso para usar la administración de derechos de Azure AD en Graph

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Administración de derechos
 
Un nuevo permiso delegado EntitlementManagement.Read.All ahora está disponible para su uso con la API de administración de derechos en la versión beta de Microsoft Graph. Para obtener más información sobre las API disponibles, vea [Trabajar con la API de administración de derechos de Azure AD](/graph/api/resources/entitlementmanagement-root?view=graph-rest-beta&preserve-view=true).

---

### <a name="identity-protection-apis-available-in-v10"></a>API de Identity Protection disponibles en la versión 1.0

**Tipo:** Nueva característica  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Las API riskyUsers y riskDetections de Microsoft Graph ahora están disponibles con carácter general. Ahora que están disponibles en el punto de conexión v 1.0, lo invitamos a utilizarlos en producción. Para más información, vea la [documentación de Microsoft Graph](/graph/api/resources/identityprotectionroot).
 
---

### <a name="sensitivity-labels-to-apply-policies-to-microsoft-365-groups-is-now-generally-available"></a>Las etiquetas de confidencialidad para aplicar directivas a grupos de Microsoft 365 ya están disponibles con carácter general

**Tipo:** Nueva característica  
**Categoría del servicio:** Administración de grupos  
**Funcionalidad del producto:** Colaboración
 

Ahora puede crear etiquetas de confidencialidad y usar la configuración de la etiqueta para aplicar directivas a grupos de Microsoft 365, incluida la directiva de privacidad (pública o privada) y la directiva de acceso de usuarios externos. Puede crear una etiqueta con la directiva de privacidad para que sea privada y con la directiva de acceso de usuarios externos para no permitir que se agreguen usuarios invitados. Cuando un usuario aplica esta etiqueta a un grupo, este será privado y no se permitirá agregar usuarios invitados al grupo. 

Las etiquetas de confidencialidad son importantes para proteger los datos críticos para la empresa y permiten administrar grupos a escala, de manera compatible y segura. Para obtener orientación sobre el uso de etiquetas de confidencialidad, vea [Asignación de etiquetas de confidencialidad a grupos de Microsoft 365 en Azure Active Directory (versión preliminar)](../enterprise-users/groups-assign-sensitivity-labels.md).
 
---

### <a name="updates-to-support-for-microsoft-identity-manager-for-azure-ad-premium-customers"></a>Actualizaciones de compatibilidad con Microsoft Identity Manager para clientes de Azure AD Premium

**Tipo:** Característica modificada  
**Categoría del servicio:** Microsoft Identity Manager  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
El Soporte técnico de Azure ya está disponible para los componentes de integración de Azure AD de Microsoft Identity Manager 2016, hasta el final del soporte extendido para Microsoft Identity Manager 2016. Para más información, vea [Actualización de soporte técnico para clientes de Azure AD Premium que usan Microsoft Identity Manager](/microsoft-identity-manager/support-update-for-azure-active-directory-premium-customers).

---

### <a name="the-use-of-group-membership-conditions-in-sso-claims-configuration-is-increased"></a>Aumento del uso de condiciones de pertenencia a grupos en la configuración de notificaciones de SSO

**Tipo:** Característica modificada  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO
 
Anteriormente, el número de grupos que podía usar al cambiar condicionalmente las notificaciones basadas en la pertenencia a grupos dentro de cualquier configuración de aplicación única estaba limitado a 10. El uso de las condiciones de pertenencia a grupos en la configuración de notificaciones de SSO ahora se ha aumentado a un máximo de 50 grupos. Para más información sobre cómo configurar las notificaciones, vea [Configuración de notificaciones de SSO en aplicaciones empresariales](../develop/active-directory-saml-claims-customization.md#emitting-claims-based-on-conditions). 

---

### <a name="enabling-basic-formatting-on-the-sign-in-page-text-component-in-company-branding"></a>Habilitar el formato básico en el componente de texto de la página de inicio de sesión para la personalización de marca de empresas

**Tipo:** Característica modificada  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
La funcionalidad de personalización de marca de la empresa en la experiencia de inicio de sesión de Azure AD/Microsoft 365 se ha actualizado para que el cliente pueda agregar hipervínculos y formatos sencillos, como negrita, subrayado y cursiva. Para obtener orientación sobre el uso de esta funcionalidad, vea [Incorporación de la personalización de marca en la página de inicio de sesión de Azure Active Directory de la organización](./customize-branding.md).

---

### <a name="provisioning-performance-improvements"></a>Mejoras en el rendimiento de aprovisionamiento

**Tipo:** Característica modificada  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
El servicio de aprovisionamiento se ha actualizado para reducir el tiempo que tarda un [ciclo incremental](../app-provisioning/how-provisioning-works.md#incremental-cycles) en completarse. Esto significa que los usuarios y grupos se aprovisionarán en sus aplicaciones más rápido que antes. Todos los nuevos trabajos de aprovisionamiento creados después del 10/6/2020 se beneficiarán automáticamente de las mejoras de rendimiento. Las aplicaciones configuradas para el aprovisionamiento antes del 10/6/2020 deberán reiniciarse una vez después del 10/6/2020 para beneficiarse de las mejoras de rendimiento. 

---

### <a name="announcing-the-deprecation-of-adal-and-ms-graph-parity"></a>Anuncio de desuso de la paridad de ADAL y MS Graph

**Tipo:** Obsoleto  
**Categoría del servicio:** N/D  
**Funcionalidad del producto:** Administración del ciclo de vida de dispositivos

Ahora que las bibliotecas de autenticación de Microsoft (MSAL) están disponibles, ya no se agregarán nuevas características a las bibliotecas de autenticación de Azure Active Directory (ADAL) y las revisiones de seguridad finalizarán el 30 de junio de 2022. Para más información sobre cómo migrar a MSAL, vea [Migración de aplicaciones a la Biblioteca de autenticación de Microsoft (MSAL)](../develop/msal-migration.md).

Además, hemos finalizado el trabajo para que toda la funcionalidad de Azure AD Graph esté disponible mediante MS Graph. Por lo tanto, las Graph API de Azure AD solo recibirán correcciones de errores y de seguridad hasta el 30 de junio de 2022. Para más información, vea [Actualización de las aplicaciones para usar la Biblioteca de autenticación de Microsoft y Microsoft Graph API](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/update-your-applications-to-use-microsoft-authentication-library/ba-p/1257363).
 
---
## <a name="may-2020"></a>Mayo de 2020

### <a name="retirement-of-properties-in-signins-riskyusers-and-riskdetections-apis"></a>Retirada de propiedades en las API signIns, riskyUsers y riskDetections

**Tipo:** Plan de cambio  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Actualmente, los tipos enumerados se usan para representar la propiedad riskType en riskDetections API y riskyUserHistoryItem (en versión preliminar). Los tipos enumerados también se usan para la propiedad riskEventTypes de signIns API. En el futuro, se representarán estas propiedades como cadenas. 

Los clientes deben realizar la transición a la propiedad riskEventType de la versión beta de riskDetections y riskyUserHistoryItem API, así como a la propiedad riskEventTypes_v2 de la versión beta de signIns API antes del 9 de septiembre de 2020. En esa fecha, se retirarán las propiedades actuales riskType y riskEventTypes. Para más información, vea [Cambios en las propiedades de eventos de riesgo y en las API de Identity Protection en Microsoft Graph](https://developer.microsoft.com/graph/blogs/changes-to-risk-event-properties-and-identity-protection-apis-on-microsoft-graph/).

--- 

### <a name="deprecation-of-riskeventtypes-property-in-signins-v10-api-on-microsoft-graph"></a>Desuso de la propiedad riskEventTypes en signIns v1.0 API en Microsoft Graph

**Tipo:** Plan de cambio  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Los tipos enumerados cambiarán a tipos de cadena al representar las propiedades de evento de riesgo en Microsoft Graph en septiembre de 2020. Además de afectar a las API en versión preliminar, este cambio también afectará a signIns API en producción.

Hemos introducido una nueva propiedad riskEventsTypes_v2 (cadena) en signIns v1.0 API. Se retirará la propiedad riskEventTypes (enumeración) actual el 11 de junio de 2022 de acuerdo con la directiva de desuso de Microsoft Graph. Los clientes deben realizar la transición a la propiedad riskEventTypes_v2 de v1.0 signIns API antes del 11 de junio de 2022. Para más información, vea [Desuso de la propiedad riskEventTypes en signIns v1.0 API en Microsoft Graph](https://developer.microsoft.com/graph/blogs/deprecation-of-riskeventtypes-property-in-signins-v1-0-api-on-microsoft-graph//).

--- 

### <a name="upcoming-changes-to-mfa-email-notifications"></a>Próximos cambios en las notificaciones por correo electrónico de MFA

**Tipo:** Plan de cambio  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 

Vamos a realizar los siguientes cambios en las notificaciones por correo electrónico de MFA en la nube:

Las notificaciones por correo electrónico se enviarán desde las siguientes direcciones: azure-noreply@microsoft.com y msonlineservicesteam@microsoftonline.com. Estamos actualizando el contenido de los correos electrónicos de alerta sobre fraudes a fin de explicar mejor los pasos necesarios para desbloquear los usos.

---

### <a name="new-self-service-sign-up-for-users-in-federated-domains-who-cant-access-microsoft-teams-because-they-arent-synced-to-azure-active-directory"></a>Nuevo registro de autoservicio para usuarios de dominios federados que no pueden acceder a Microsoft Teams porque no están sincronizados con Azure Active Directory.

**Tipo:** Plan de cambio  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 

Actualmente, los usuarios que están en dominios federados en Azure AD, pero que no están sincronizados en el inquilino, no pueden acceder a Teams. A partir de finales de junio, esta nueva funcionalidad les permitirá hacerlo gracias a la extensión de la característica de registro con verificación por correo electrónico. Esto permitirá a los usuarios iniciar sesión en un proveedor de identidades (IdP) federado. En el caso de los usuarios que aún no tengan un objeto de usuario en Azure ID, el objeto se creará automáticamente y se autenticará para Teams. El objeto de usuario se marcará como "registro de autoservicio". Se trata de una extensión de la característica existente para que los usuarios de los dominios administrados puedan registrarse mediante la verificación por correo electrónico y se pueda llevar un control utilizando la misma marca. La implementación de este cambio se llevará a cabo durante los dos próximos meses. Vea las actualizaciones de la documentación [aquí](../enterprise-users/directory-self-service-signup.md).
 
---

### <a name="upcoming-fix-the-oidc-discovery-document-for-the-azure-government-cloud-is-being-updated-to-reference-the-correct-graph-endpoints"></a>Próxima corrección: El documento de detección de OIDC para la nube de Azure Government se está actualizando para que haga referencia a los puntos de conexión de Graph correctos.

**Tipo:** Plan de cambio  
**Categoría del servicio:** Nubes soberanas  
**Funcionalidad del producto:** Autenticación de usuarios
 
A partir de junio, el documento de detección de OIDC [Plataforma de identidad de Microsoft y protocolo OpenID Connect](../develop/v2-protocols-oidc.md) sobre el punto de conexión de la [nube de Azure Government](../develop/authentication-national-cloud.md) (login.microsoftonline.us), comenzará a devolver el punto de conexión de [Graph de la nube nacional ](/graph/deployments) correcto (https://graph.microsoft.us o https://dod-graph.microsoft.us), según el inquilino proporcionado).  Actualmente, proporciona el campo "msgraph_host" del punto de conexión de Graph incorrecto (graph.microsoft.com).  

Esta corrección de errores se implementará gradualmente durante 2 meses aproximadamente.  

---

### <a name="azure-government-users-will-no-longer-be-able-to-sign-in-on-loginmicrosoftonlinecom"></a>Los usuarios de Azure Government ya no podrán iniciar sesión en login.microsoftonline.com

**Tipo:** Plan de cambio  
**Categoría del servicio:** Nubes soberanas  
**Funcionalidad del producto:** Autenticación de usuarios
 
El 1 de junio de 2018, la autoridad oficial de Azure Active Directory (Azure AD) para Azure Government cambió de https://login-us.microsoftonline.com a https://login.microsoftonline.us. Si posee una aplicación en un inquilino de Azure Government, tiene que actualizarla para que los usuarios inicien sesión en el punto de conexión .us.

El 5 de mayo, Azure AD comenzó a aplicar el cambio relativo al punto de conexión, por lo que los usuarios de Azure Government no podrán iniciar sesión en aplicaciones hospedadas en inquilinos de Azure Government mediante el punto de conexión público (microsoftonline.com). Las aplicaciones afectadas comenzarán a obtener el error AADSTS900439 - USGClientNotSupportedOnPublicEndpoint. 

Este cambio se implementará de forma gradual y se espera que se haya completado para todas las aplicaciones en junio de 2020. Para más información, consulte la [entrada de blog sobre Azure Government](https://devblogs.microsoft.com/azuregov/azure-government-aad-authority-endpoint-update/).

---

### <a name="saml-single-logout-request-now-sends-nameid-in-the-correct-format"></a>La solicitud de cierre de sesión único de SAML ahora envía el valor de NameID con el formato correcto

**Tipo:** Corregido  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
Cuando un usuario hace clic para cerrar sesión (por ejemplo, en el portal Mis aplicaciones), Azure AD envía un mensaje de cierre de sesión único de SAML a cada aplicación que esté activa en la sesión de usuario y tenga configurada una dirección URL de cierre de sesión. Estos mensajes contienen un valor de NameID con formato persistente.

Si el token de inicio de sesión de SAML original utilizaba un formato diferente para NameID (por ejemplo, correo electrónico o UPN), la aplicación SAML no podía correlacionar el valor de NameID en el mensaje de cierre de sesión para una sesión existente, dado que los valores de NameID utilizados en ambos mensajes eran diferentes. Esto hacía que la aplicación SAML descartara el mensaje de cierre de sesión y que la sesión del usuario se mantuviera activa. Esta corrección hace que el mensaje de cierre de sesión sea coherente con el valor de NameID configurado para la aplicación.

---

### <a name="hybrid-identity-administrator-role-is-now-available-with-cloud-provisioning"></a>El rol de administrador de identidades híbridas ahora está disponible con aprovisionamiento en la nube

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento en la nube de Azure AD  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
Los administradores de TI pueden empezar a usar el nuevo rol de "administrador de identidades híbridas" como el rol con privilegios mínimos para configurar el aprovisionamiento en la nube de Azure AD Connect. Con este nuevo rol, ya no tiene que usar el rol de administrador global para instalar y configurar el aprovisionamiento en la nube. [Más información](../roles/delegate-by-task.md#connect).
 
---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---may-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD, mayo de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En mayo de 2020, hemos agregado 36 nuevas aplicaciones a la galería de aplicaciones con compatibilidad de federación:

[Moula](https://moula.com.au/pay/merchants), [Surveypal](https://www.surveypal.com/app), [Kbot365](https://www.konverso.ai/virtual-assistant-digital-workplace/), [TackleBox](http://www.tacklebox.app/), [Powell Teams](https://powell-software.com/en/powell-teams-en/), [Talentsoft Assistant](https://msteams.talent-soft.com/), [ASC Recording Insights](https://teams.asc-recording.app/product), [GO1](https://www.go1.com/), [B-Engaged](https://b-engaged.se/), [Competella Contact Center Workgroup](http://www.competella.com/), [Asite](http://www.asite.com/), [ImageSoft Identity](https://identity.imagesoftinc.com/), [My IBISWorld](https://identity.imagesoftinc.com/), [insuite](../saas-apps/insuite-tutorial.md), [Change Process Management](../saas-apps/change-process-management-tutorial.md), [Cyara CX Assurance Platform](../saas-apps/cyara-cx-assurance-platform-tutorial.md), [Smart Global Governance](../saas-apps/smart-global-governance-tutorial.md), [Prezi](../saas-apps/prezi-tutorial.md), [Mapbox](../saas-apps/mapbox-tutorial.md), [Datava Enterprise Service Platform](../saas-apps/datava-enterprise-service-platform-tutorial.md), [Whimsical](../saas-apps/whimsical-tutorial.md), [Trelica](../saas-apps/trelica-tutorial.md), [EasySSO for Confluence](../saas-apps/easysso-for-confluence-tutorial.md), [EasySSO for BitBucket](../saas-apps/easysso-for-bitbucket-tutorial.md), [EasySSO for Bamboo](../saas-apps/easysso-for-bamboo-tutorial.md), [Torii](../saas-apps/torii-tutorial.md), [Axiad Cloud](../saas-apps/axiad-cloud-tutorial.md), [Humanage](../saas-apps/humanage-tutorial.md), [ColorTokens ZTNA](../saas-apps/colortokens-ztna-tutorial.md), [CCH Tagetik](../saas-apps/cch-tagetik-tutorial.md), [ShareVault](../saas-apps/sharevault-tutorial.md), [Vyond](../saas-apps/vyond-tutorial.md), [TextExpander](../saas-apps/textexpander-tutorial.md), [Anyone Home CRM](../saas-apps/anyone-home-crm-tutorial.md), [askSpoke](../saas-apps/askspoke-tutorial.md), [ice Contact Center](../saas-apps/ice-contact-center-tutorial.md)

La documentación de todas las aplicaciones está disponible aquí: https://aka.ms/AppsTutorial

Para mostrar su aplicación en la galería de aplicaciones Azure AD, lea los detalles aquí: https://aka.ms/AzureADAppRequest

---

### <a name="report-only-mode-for-conditional-access-is-now-generally-available"></a>El modo Solo informe para el Acceso condicional ya está disponible con carácter general.

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad

El [modo Solo informe para el Acceso condicional de Azure AD](../conditional-access/concept-conditional-access-report-only.md) le permite evaluar el resultado de una directiva sin necesidad de aplicar controles de acceso. Puede probar las directivas del modo Solo informe en toda la organización y comprender su impacto antes de habilitarlas, lo que hará que la implementación sea más segura y fácil. En los últimos meses, hemos observado una importante adopción del modo de solo informe, y más de 26 millones de usuarios están incluidos en el ámbito de la directiva de solo informe. Con el anuncio de hoy, se crearán nuevas directivas de acceso condicional de Azure AD en el modo de solo informe de forma predeterminada. Esto significa que puede supervisar el impacto de las directivas desde el momento en el que se crean. Además, aquellos usuarios que utilizan Microsoft Graph API, también pueden [administrar directivas de solo informe mediante programación](/graph/api/resources/conditionalaccesspolicy?view=graph-rest-beta&preserve-view=true). 

---

### <a name="self-service-sign-up-for-guest-users"></a>Registro de autoservicio para usuarios invitados

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Mediante las identidades externas en Azure AD, puede facilitar a personas ajenas a la organización el acceso a sus aplicaciones y recursos, además de permitirles iniciar sesión con la identidad que prefieran. Al compartir una aplicación con usuarios externos, es posible que no siempre sepa de antemano quién necesitará tener acceso a la aplicación. Con el [registro de autoservicio](../external-identities/self-service-sign-up-overview.md), puede facilitar a los usuarios invitados el registro y el acceso a una cuenta de invitado para sus aplicaciones de línea de negocio (LOB). El flujo de registro se puede crear y personalizar para admitir Azure AD e identidades sociales. También puede recopilar información adicional sobre el usuario durante el registro.

---

 ### <a name="conditional-access-insights-and-reporting-workbook-is-generally-available"></a>El libro Conditional Access Insights and Reporting (Información detallada e informes del acceso condicional) está disponible con carácter general

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad

El [libro Conditional Access Insights and Reporting (Información detallada e informes del acceso condicional)](../conditional-access/howto-conditional-access-insights-reporting.md) ofrece a los administradores un resumen del acceso condicional de Azure AD en su inquilino. Con la funcionalidad de seleccionar una directiva individual, los administradores pueden comprender mejor lo que hace cada directiva y supervisar los cambios en tiempo real. El libro transmite los datos almacenados en Azure Monitor, que puede configurar en unos minutos [según estas instrucciones](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md). Para encontrar el panel con más facilidad, lo hemos trasladado a la nueva pestaña Conclusiones e informes del menú Acceso condicional de Azure AD.

---

### <a name="policy-details-blade-for-conditional-access-is-in-public-preview"></a>La hoja Detalles de la directiva de Acceso condicional está en versión preliminar pública.

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad

La nueva [hoja Detalles de la directiva](../conditional-access/troubleshoot-conditional-access.md) muestra las asignaciones, condiciones y controles que se han llevado a cabo durante la evaluación de la directiva de acceso condicional. Para acceder a la hoja, seleccione una fila en las pestañas Acceso condicional o Solo informe, en los detalles de inicio de sesión.

---

### <a name="new-query-capabilities-for-directory-objects-in-microsoft-graph-are-in-public-preview"></a>Nuevas funciones de consulta para objetos de directorio en Microsoft Graph ahora disponibles en versión preliminar pública

**Tipo:** Nueva característica  
**Categoría del servicio:** MS Graph **Funcionalidad del producto:** Experiencia para el desarrollador

Se están incorporando nuevas funciones a las API de objetos de directorio de Microsoft Graph, que permiten realizar operaciones de recuento, búsqueda, filtrado y ordenación. Esto permitirá a los desarrolladores consultar rápidamente nuestros objetos de directorio sin necesidad de buscar alternativas como la ordenación y el filtrado en memoria. Puede obtener más información en esta [entrada de blog](https://aka.ms/CountFilterMSGraphAAD).

Actualmente nos encontramos en la fase de versión preliminar pública, a la espera de comentarios. Envíe sus comentarios por medio de esta [breve encuesta](https://aka.ms/MsGraphAADSurveyDocs).

---

### <a name="configure-saml-based-single-sign-on-using-microsoft-graph-api-beta"></a>Configuración del inicio de sesión único basado en SAML mediante Microsoft Graph API (beta)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO
 
Ya se ofrece compatibilidad para crear y configurar una aplicación desde la galería de Azure AD mediante MS Graph API en la versión beta. Si necesita configurar el inicio de sesión único basado en SAML para varias instancias de una aplicación, ahorre tiempo y utilice Microsoft Graph API para [automatizar la configuración del inicio de sesión único basado en SAML](/graph/application-saml-sso-configure-api).
 
---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---may-2020"></a>Nuevos conectores de aprovisionamiento disponibles en la galería de aplicaciones de Azure AD, mayo de 2020

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Integración de terceros
 
Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

* [8x8](../saas-apps/8x8-provisioning-tutorial.md)
* [Juno Journey](../saas-apps/juno-journey-provisioning-tutorial.md)
* [MediusFlow](../saas-apps/mediusflow-provisioning-tutorial.md)
* [New Relic by Organization](../saas-apps/new-relic-by-organization-provisioning-tutorial.md)
* [Oracle Cloud Infrastructure Console](../saas-apps/oracle-cloud-infrastructure-console-provisioning-tutorial.md)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

---

### <a name="saml-token-encryption-is-generally-available"></a>El cifrado de tokens SAML está disponible con carácter general

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO
 
El [cifrado de tokens SAML](../manage-apps/howto-saml-token-encryption.md) permite configurar las aplicaciones para que reciban aserciones de SAML cifradas. La característica ya está disponible con carácter general en todas las nubes.
 
---

### <a name="group-name-claims-in-application-tokens-is-generally-available"></a>Las notificaciones de nombre de grupo en tokens de aplicación están disponibles con carácter general

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO
 
Las notificaciones de grupo emitidas en un token ahora se pueden limitar a los grupos asignados a la aplicación.  Esto es especialmente importante cuando los usuarios son miembros de un gran número de grupos y había riesgo de superar los límites de tamaño del token. Esta nueva funcionalidad permite [agregar nombres de grupo a los tokens](../hybrid/how-to-connect-fed-group-claims.md) con carácter general.
 
---

### <a name="workday-writeback-now-supports-setting-work-phone-number-attributes"></a>La escritura diferida de Workday ahora admite el establecimiento de atributos relativos al número de teléfono del trabajo

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
Hemos mejorado la aplicación de aprovisionamiento de escritura diferida de Workday para que admita la escritura diferida de los atributos relativos al número de teléfono del trabajo y al número de móvil. Además del correo electrónico y el nombre de usuario, ahora puede configurar la aplicación de aprovisionamiento de escritura diferida de Workday para transferir los valores de número de teléfono de Azure AD a Workday. Para más información sobre cómo configurar la escritura diferida de números de teléfono, consulte el tutorial de la aplicación de [escritura diferida de Workday](../saas-apps/workday-writeback-tutorial.md). 

---

### <a name="publisher-verification-preview"></a>Comprobación del publicador (versión preliminar)

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Experiencia para el desarrollador
 
La verificación del publicador (versión preliminar) ayuda a los administradores y usuarios finales a comprender la autenticidad de los desarrolladores de aplicaciones que se integran con la plataforma de identidad de Microsoft. Para más información, consulte [Comprobación del publicador (versión preliminar)](../develop/publisher-verification-overview.md).
 
---

### <a name="authorization-code-flow-for-single-page-apps"></a>Flujo de código de autorización para aplicaciones de una sola página

**Tipo:** Característica modificada **Categoría del servicio:** Autenticación **Funcionalidad del producto:** Experiencia para el desarrollador

Debido a las [restricciones de cookies de terceros como Safari ITP](../develop/reference-third-party-cookies-spas.md) aplicadas en el explorador, las aplicaciones de página única (SPA) tendrán que usar el flujo de código de autorización en lugar del flujo implícito para mantener el SSO. MSAL.js v2.x admite ahora el flujo de código de autorización. Las actualizaciones correspondientes están disponibles en Azure Portal. De este modo, podrá actualizar sus aplicaciones de página única para que sean del tipo "spa" y utilicen el flujo de código de autenticación. Para más instrucciones, consulte [Inicio rápido: Inicio de sesión de los usuarios y obtención de un token de acceso en una SPA de JavaScript mediante el flujo de código de autorización](../develop/quickstart-v2-javascript-auth-code.md).

---

### <a name="improved-filtering-for-devices-is-in-public-preview"></a>El filtrado mejorado para dispositivos está en versión preliminar pública

**Tipo:** Característica modificada   
**Categoría del servicio:** Administración de dispositivos **Funcionalidad del producto:** Administración del ciclo de vida de dispositivos
 
Antes, los únicos filtros que se podían usar eran "Habilitado" y "Fecha de actividad". Ahora, puede [filtrar la lista de dispositivos en función de más propiedades](../devices/device-management-azure-portal.md#device-list-filtering-preview), como el tipo de sistema operativo, el tipo de combinación, el cumplimiento, etc. Estas incorporaciones permitirán encontrar fácilmente un dispositivo determinado.

---

### <a name="the-new-app-registrations-experience-for-azure-ad-b2c-is-now-generally-available"></a>La nueva experiencia Registros de aplicaciones para Azure AD B2C ya está disponible con carácter general

**Tipo:** Característica modificada   
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
La nueva experiencia Registros de aplicaciones para Azure AD B2C ya está disponible con carácter general. 

Antes, había que administrar las aplicaciones Azure AD B2C orientadas al consumidor con independencia del resto de aplicaciones mediante la experiencia "Aplicaciones" heredada. Eso implicaba distintas experiencias de creación de aplicaciones en distintas ubicaciones de Azure.

La nueva experiencia muestra todos los registros de aplicaciones de Azure AD B2C y Azure AD en un solo lugar y proporciona una manera coherente de administrarlos. Si necesita administrar una aplicación orientada al cliente o una aplicación que tenga acceso a Microsoft Graph para administrar recursos de Azure AD B2C mediante programación, solo tendrá que aprender una manera de hacer cosas.

Puede acceder a la nueva experiencia desde el servicio Azure AD B2C. Una vez ahí, seleccione la hoja Registros de aplicaciones. También puede acceder a la experiencia desde el servicio de Azure Active Directory.

La experiencia Registros de aplicaciones de Azure AD B2C se basa en la [experiencia de registro de aplicaciones](https://developer.microsoft.com/identity/blogs/new-app-registrations-experience-is-now-generally-available/) general para inquilinos de Azure AD, pero se ha adaptado para Azure AD B2C. La experiencia "Aplicaciones" heredada dejará de utilizarse en el futuro.

Para más información, visite [ La nueva experiencia Registros de aplicaciones para Azure Active Directory B2C ](../../active-directory-b2c/app-registrations-training-guide.md).

---
## <a name="april-2020"></a>Abril de 2020

### <a name="combined-security-info-registration-experience-is-now-generally-available"></a>La experiencia combinada del registro de la información de seguridad ya está disponible con carácter general.

**Tipo:** Nueva característica

**Categoría del servicio:** Autenticaciones (inicios de sesión)

**Funcionalidad del producto:** Seguridad y protección de la identidad

La experiencia de registro combinada con autenticación multifactor (MFA) y autoservicio de restablecimiento de contraseña (SSPR) ahora están disponibles con carácter general. Esta nueva experiencia de registro permite a los usuarios registrarse en MFA y SSPR en un único proceso paso a paso. Al implementar la nueva experiencia para la organización, los usuarios pueden registrarse en menos tiempo y con menos complicaciones. Consulte la entrada de blog [aquí](https://bit.ly/3etiRyQ).

---

### <a name="continuous-access-evaluation"></a>Evaluación continua de acceso

**Tipo:** Nueva característica

**Categoría del servicio:** Autenticaciones (inicios de sesión)

**Funcionalidad del producto:** Seguridad y protección de la identidad

Evaluación continua de acceso es una nueva característica de seguridad que permite el cumplimiento casi en tiempo real de las directivas en los usuarios de confianza que consumen tokens de acceso de Azure AD cuando se producen eventos en Azure AD (por ejemplo, la eliminación de cuentas de usuario). Esta característica la vamos a implementar en primer lugar para clientes de Teams y Outlook. Para más información, consulte el [blog](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/moving-towards-real-time-policy-and-security-enforcement/ba-p/1276933) y la [documentación](../conditional-access/concept-continuous-access-evaluation.md).

---

### <a name="sms-sign-in-firstline-workers-can-sign-in-to-azure-ad-backed-applications-with-their-phone-number-and-no-password"></a>Inicio de sesión por SMS: Los trabajadores de primera línea pueden iniciar sesión en las aplicaciones respaldadas por Azure AD con su número de teléfono y sin contraseña.

**Tipo:** Nueva característica

**Categoría del servicio:** Autenticaciones (inicios de sesión)

**Funcionalidad del producto:** Autenticación de usuarios

Office está lanzando una serie de aplicaciones empresariales orientadas a móviles dirigidas a organizaciones no tradicionales y a empleados de grandes organizaciones que no usan el correo electrónico como método de comunicación principal. Estas aplicaciones se dirigen a empleados de primera línea, trabajadores sin escritorio, agentes de campo o empleados de comercios minoristas que podrían no recibir una dirección de correo electrónico de su empleador, tener acceso a un equipo o a recursos de TI. Este proyecto permitirá a estos empleados iniciar sesión en las aplicaciones empresariales solo con un número de teléfono y la devolución de un código. Para más información, consulte la [documentación del administrador](../authentication/howto-authentication-sms-signin.md) y la [documentación del usuario final](../user-help/sms-sign-in-explainer.md).

---

### <a name="invite-internal-users-to-use-b2b-collaboration"></a>Invitar a usuarios internos a usar la colaboración B2B

**Tipo:** Nueva característica

**Categoría del servicio:** B2B

**Funcionalidad del producto:**

Vamos a expandir la funcionalidad de invitación B2B para que se pueda invitar a las cuentas internas existentes a usar las credenciales de colaboración B2B en adelante. Esto se consigue pasando el objeto de usuario a la API de invitación, además de los parámetros típicos, como la dirección de correo electrónico invitada. El identificador de objeto del usuario, su UPN, la pertenencia a grupos, la asignación de aplicaciones, entre otros, permanecen intactos, pero en adelante usarán B2B para autenticarse con las credenciales de su inquilino particular en lugar de con las credenciales internas que usaban antes de la invitación. Para más información, consulte la [documentación](../external-identities/invite-internal-users.md).

---

### <a name="report-only-mode-for-conditional-access-is-now-generally-available"></a>El modo Solo informe para el Acceso condicional ya está disponible con carácter general.

**Tipo:** Nueva característica

**Categoría del servicio:** Acceso condicional

**Funcionalidad del producto:** Seguridad y protección de la identidad

El [modo Solo informe para el Acceso condicional de Azure AD](../conditional-access/concept-conditional-access-report-only.md) le permite evaluar el resultado de una directiva sin necesidad de aplicar controles de acceso. Puede probar las directivas del modo Solo informe en toda la organización y comprender su impacto antes de habilitarlas, lo que hará que la implementación sea más segura y fácil. Durante los últimos meses, hemos visto una importante adopción del modo Solo informe, con más de 26 millones de usuarios. Con este anuncio, se crearán nuevas directivas de acceso condicional de Azure AD en modo Solo informe de forma predeterminada. Esto significa que puede supervisar el impacto de las directivas desde el momento en el que se crean. Además, para aquellos usuarios que utilizan Microsoft Graph API, también puede [administrar directivas de Solo informe mediante programación](/graph/api/resources/conditionalaccesspolicy?view=graph-rest-beta&preserve-view=true). 

---

### <a name="conditional-access-insights-and-reporting-workbook-is-generally-available"></a>El libro Conclusiones e informes de Acceso condicional está disponible con carácter general.

**Tipo:** Nueva característica

**Categoría del servicio:** Acceso condicional

**Funcionalidad del producto:** Seguridad y protección de la identidad

El [libro Conclusiones e informes](../conditional-access/howto-conditional-access-insights-reporting.md) de Acceso condicional proporciona a los administradores una vista resumida del acceso condicional de Azure AD en el inquilino. Con la funcionalidad de seleccionar una directiva individual, los administradores pueden comprender mejor lo que hace cada directiva y supervisar los cambios en tiempo real. El libro transmite los datos almacenados en Azure Monitor, que puede configurar en unos minutos [según estas instrucciones](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md). Para encontrar el panel con más facilidad, lo hemos trasladado a la nueva pestaña Conclusiones e informes del menú Acceso condicional de Azure AD.

---

### <a name="policy-details-blade-for-conditional-access-is-in-public-preview"></a>La hoja Detalles de la directiva de Acceso condicional está en versión preliminar pública.

**Tipo:** Nueva característica

**Categoría del servicio:** Acceso condicional

**Funcionalidad del producto:** Seguridad y protección de la identidad

La nueva [hoja Detalles de la directiva](../conditional-access/troubleshoot-conditional-access.md) muestra qué asignaciones, condiciones y controles se cumplieron durante la evaluación de la directiva de acceso condicional. Para acceder a la hoja, seleccione una fila en las pestañas **Acceso condicional** o **Solo informe** de los detalles de inicio de sesión.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---april-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (abril de 2020)

**Tipo:** Nueva característica

**Categoría del servicio:** Aplicaciones empresariales

**Funcionalidad del producto:** Integración de terceros

En abril de 2020, hemos agregado a la galería de aplicaciones estas 31 nuevas aplicaciones que admiten federación: 

[SincroPool Apps](https://www.sincropool.com/), [SmartDB](https://hibiki.dreamarts.co.jp/smartdb/trial/), [Float](../saas-apps/float-tutorial.md), [LMS365](https://lms.365.systems/), [IWT Procurement Suite](../saas-apps/iwt-procurement-suite-tutorial.md), [Lunni](https://lunni.fi/), [EasySSO for Jira](../saas-apps/easysso-for-jira-tutorial.md), [Virtual Training Academy](https://vta.c3p.ca/app/en/openid?authenticate_with=microsoft), [Meraki Dashboard](../saas-apps/meraki-dashboard-tutorial.md), [Microsoft 365 Mover](https://app.mover.io/login), [Speaker Engage](https://speakerengage.com/login.php), [Honestly](../saas-apps/honestly-tutorial.md), [Ally](../saas-apps/ally-tutorial.md), [DutyFlow](https://app.dutyflow.nl/), [AlertMedia](../saas-apps/alertmedia-tutorial.md), [gr8 People](../saas-apps/gr8-people-tutorial.md), [Pendo](../saas-apps/pendo-tutorial.md), [HighGround](../saas-apps/highground-tutorial.md), [Harmony](../saas-apps/harmony-tutorial.md), [Timetabling Solutions](../saas-apps/timetabling-solutions-tutorial.md), [SynchroNet CLICK](../saas-apps/synchronet-click-tutorial.md), [empower](https://www.made-in-office.com/en/), [Fortes Change Cloud](../saas-apps/fortes-change-cloud-tutorial.md), [Litmus](../saas-apps/litmus-tutorial.md), [GroupTalk](https://recorder.grouptalk.com/), [Frontify](../saas-apps/frontify-tutorial.md), [MongoDB Cloud](../saas-apps/mongodb-cloud-tutorial.md), [TickitLMS Learn](../saas-apps/tickitlms-learn-tutorial.md), [COCO](https://hexaware.com/partnerships-and-alliances/digital-transformation-using-microsoft-azure/), [Nitro Productivity Suite](../saas-apps/nitro-productivity-suite-tutorial.md), [Trend Micro Web Security(TMWS)](../saas-apps/trend-micro-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="microsoft-graph-delta-query-support-for-oauth2permissiongrant-available-for-public-preview"></a>La compatibilidad con las consultas delta de Microsoft Graph para oAuth2PermissionGrant está disponible para la versión preliminar pública.

**Tipo:** Nueva característica

**Categoría del servicio:** MS Graph

**Funcionalidad del producto:** Experiencia para el desarrollador

Las consultas delta para oAuth2PermissionGrant están disponibles en versión preliminar pública. Ahora puede realizar el seguimiento de los cambios sin necesidad de sondear Microsoft Graph continuamente. [Más información.](/graph/api/oAuth2PermissionGrant-delta?tabs=http&view=graph-rest-beta&preserve-view=true)

---

### <a name="microsoft-graph-delta-query-support-for-organizational-contact-generally-available"></a>La compatibilidad con las consultas delta de Microsoft Graph para contactos de la organización está disponible con carácter general.

**Tipo:** Nueva característica

**Categoría del servicio:** MS Graph

**Funcionalidad del producto:** Experiencia para el desarrollador

Las consultas delta para contactos de la organización están disponibles con carácter general. Ahora puede realizar el seguimiento de los cambios en las aplicaciones de producción sin necesidad de sondear Microsoft Graph continuamente. Reemplace el código existente que ahora sondea continuamente los datos de orgContact por una consulta delta para mejorar significativamente el rendimiento. [Más información.](/graph/api/orgcontact-delta?tabs=http)

---

### <a name="microsoft-graph-delta-query-support-for-application-generally-available"></a>La compatibilidad con las consultas delta de Microsoft Graph para aplicaciones está disponible con carácter general.

**Tipo:** Nueva característica

**Categoría del servicio:** MS Graph

**Funcionalidad del producto:** Experiencia para el desarrollador

Las consultas delta para aplicaciones están disponibles con carácter general. Ahora puede realizar el seguimiento de los cambios en las aplicaciones de producción sin necesidad de sondear Microsoft Graph continuamente. Reemplace el código existente que ahora sondea continuamente los datos de la aplicación por una consulta delta para mejorar significativamente el rendimiento. [Más información.](/graph/api/application-delta)

---

### <a name="microsoft-graph-delta-query-support-for-administrative-units-available-for-public-preview"></a>La compatibilidad con las consultas delta de Microsoft Graph para unidades administrativas está disponible en versión preliminar pública.

**Tipo:** Nueva característica

**Categoría del servicio:** MS Graph

**Funcionalidad del producto:** Las consultas delta de la experiencia del desarrollador para unidades administrativas están disponibles en versión preliminar pública. Ahora puede realizar el seguimiento de los cambios sin necesidad de sondear Microsoft Graph continuamente. [Más información.](/graph/api/administrativeunit-delta?tabs=http&view=graph-rest-beta&preserve-view=true)

---

### <a name="manage-authentication-phone-numbers-and-more-in-new-microsoft-graph-beta-apis"></a>Administre los números de teléfono de autenticación y mucho más en las nuevas Microsoft Graph API en versión beta.

**Tipo:** Nueva característica

**Categoría del servicio:** MS Graph

**Funcionalidad del producto:** Experiencia para el desarrollador

Estas API son una herramienta fundamental para administrar los métodos de autenticación de los usuarios. Ahora puede realizar un registro previo y administrar mediante programación los autenticadores usados para MFA y el autoservicio de restablecimiento de contraseña (SSPR). Esta ha sido una de las características más solicitadas en las áreas de Azure AD MFA, SSPR y Microsoft Graph. Las nuevas API que hemos publicado en esta oleada le ofrecen la posibilidad de:

- Leer, agregar, actualizar y eliminar los teléfonos de autenticación de un usuario
- Restablecer la contraseña de un usuario
- Activar y desactivar el inicio de sesión por SMS

Para más información, consulte [Introducción a la API de métodos de autenticación de Azure AD](/graph/api/resources/authenticationmethods-overview?view=graph-rest-beta&preserve-view=true).

---

### <a name="administrative-units-public-preview"></a>Versión preliminar de las unidades administrativas

**Tipo:** Nueva característica

**Categoría del servicio:** Roles de Azure AD

**Funcionalidad del producto:** Control de acceso

Las unidades administrativas permiten conceder permisos de administrador que están restringidos a un departamento, a una región o a otro segmento de la organización que se defina. Puede usar las unidades administrativas para delegar permisos en los administradores regionales o establecer directivas de forma pormenorizada. Por ejemplo, un administrador de cuentas de usuario podría actualizar la información del perfil, restablecer las contraseñas y asignar las licencias para los usuarios solo en su unidad administrativa.

Mediante el uso de unidades administrativas, un administrador central podría:

- Crear una unidad administrativa para la administración descentralizada de los recursos.
- Asignar un rol con permisos administrativos solo sobre los usuarios de Azure AD de una unidad administrativa.
- Rellenar las unidades administrativas con usuarios y grupos según sea necesario.

Para más información, consulte [Administración de unidades administrativas en Azure Active Directory (versión preliminar)](../roles/administrative-units.md).

---

### <a name="printer-administrator-and-printer-technician-built-in-roles"></a>Roles integrados Administrador de impresoras y Técnico de impresoras

**Tipo:** Nueva característica

**Categoría del servicio:** Roles de Azure AD

**Funcionalidad del producto:** Control de acceso

**Administrador de impresoras**: los usuarios con este rol pueden registrar impresoras y administrar todos los aspectos de todas las configuraciones de impresora en la solución de impresión universal de Microsoft, incluida la configuración del conector de impresión universal. Pueden dar su consentimiento a todas las solicitudes de permiso de impresión delegada. Los administradores de impresoras también tienen acceso a los informes de impresión. 

**Técnico de impresoras**: los usuarios con este rol pueden registrar impresoras y administrar el estado de la impresora en la solución de impresión universal de Microsoft. También pueden leer toda la información del conector. Una tarea clave que un Técnico de impresoras no puede realizar es establecer permisos de usuario en las impresoras ni compartir impresoras. [Más información.](../roles/permissions-reference.md#printer-administrator)

---

### <a name="hybrid-identity-admin-built-in-role"></a>Rol integrado Administrador de identidades híbridas

**Tipo:** Nueva característica

**Categoría del servicio:** Roles de Azure AD

**Funcionalidad del producto:** Control de acceso

Los usuarios de este rol pueden habilitar, configurar y administrar servicios y configuraciones relacionados con la habilitación de la identidad híbrida en Azure AD. Este rol concede la capacidad de configurar Azure AD en uno de los tres métodos de autenticación admitidos: la sincronización de hash de contraseña (PHS), la autenticación de paso a través (PTA) o la federación (AD FS o proveedor de federación de terceros), así como implementar la infraestructura local relacionada para habilitarlos. La infraestructura local incluye agentes de aprovisionamiento y PTA. Este rol concede la capacidad de habilitar el inicio de sesión único de conexión directa (S-SSO) para habilitar la autenticación directa en dispositivos que no son de Windows 10 o equipos que no son de Windows Server 2016. Además, este rol concede la capacidad de ver los registros de inicio de sesión y el acceso a los datos de mantenimiento y análisis para la supervisión y la solución de problemas. [Más información.](../roles/permissions-reference.md#hybrid-identity-administrator)

---

### <a name="network-administrator-built-in-role"></a>Rol integrado Administrador de red

**Tipo:** Nueva característica

**Categoría del servicio:** Roles de Azure AD

**Funcionalidad del producto:** Control de acceso

Los usuarios con este rol pueden revisar las recomendaciones de la arquitectura de perímetro de red de Microsoft que se basan en la telemetría de red desde sus ubicaciones de usuario. El rendimiento de red para Microsoft 365 se basa en una cuidadosa arquitectura de perímetro de red de cliente empresarial que suele ser específica de la ubicación del usuario. Este rol permite la edición de ubicaciones de usuario detectadas y la configuración de parámetros de red para esas ubicaciones con el fin de facilitar la mejora de las medidas de telemetría y las recomendaciones de diseño. [Más información.](../roles/permissions-reference.md#network-administrator)

---

### <a name="bulk-activity-and-downloads-in-the-azure-ad-admin-portal-experience"></a>Actividad y descargas masivas en la experiencia del portal de administración de Azure AD

**Tipo:** Nueva característica

**Categoría del servicio:** User Management

**Funcionalidad del producto:** Directorio

Ahora puede realizar actividades masivas con usuarios y grupos de Azure AD mediante la carga de un archivo CSV en la experiencia del portal de administración de Azure AD. Puede crear usuarios o eliminar usuarios, e invitar a usuarios. También puede agregar y quitar miembros de un grupo.

Además, puede descargar listas de recursos de Azure AD desde la experiencia del portal de administración de Azure AD. Puede descargar la lista de usuarios del directorio, la lista de grupos del directorio y los miembros de un grupo determinado.

Para más información, consulte lo siguiente:

- [Crear usuarios](../enterprise-users/users-bulk-add.md) o [invitar a usuarios ](../external-identities/tutorial-bulk-invite.md).
- [Eliminar usuarios](../enterprise-users/users-bulk-delete.md) o [restaurar usuarios eliminados](../enterprise-users/users-bulk-restore.md).
- [Descargar la lista de usuarios](../enterprise-users/users-bulk-download.md) o [descargar la lista de grupos](../enterprise-users/groups-bulk-download.md)
- [Agregar (importar) miembros](../enterprise-users/groups-bulk-import-members.md), [eliminar miembros](../enterprise-users/groups-bulk-remove-members.md) o [descargar la lista de miembros](../enterprise-users/groups-bulk-download-members.md) de un grupo.

---

### <a name="my-staff-delegated-user-management"></a>Administración delegada de usuarios Mi personal

**Tipo:** Nueva característica

**Categoría del servicio:** User Management

**Funcionalidad del producto:**

Mi personal permite a los administradores de primera línea, como el administrador de una tienda, garantizar que los miembros del personal puedan acceder a sus cuentas de Azure AD. En lugar de depender de un departamento de soporte técnico central, las organizaciones pueden delegar tareas comunes, como el restablecimiento de contraseñas o el cambio de números de teléfono, en un administrador de primera línea. Con Mi personal, un usuario que no puede acceder a su cuenta puede recuperar el acceso con tan solo un par de clics, sin necesidad de ayuda del personal de TI o el departamento de soporte técnico. Para más información, consulte [Administración de usuarios con Mi personal (versión preliminar)](../roles/my-staff-configure.md) y [Delegación de la administración de usuarios con Mi personal (versión preliminar)](../user-help/my-staff-team-manager.md).

---

### <a name="an-upgraded-end-user-experience-in-access-reviews"></a>Una experiencia de usuario final actualizada en las revisiones de acceso

**Tipo:** Característica modificada

**Categoría del servicio:** Revisiones de acceso

**Funcionalidad del producto:** Identity Governance

Hemos actualizado la experiencia del revisor para las revisiones de acceso de Azure AD en el portal Aplicaciones. A finales de abril, los revisores que han iniciado sesión en la experiencia del revisor de las revisiones de acceso de Azure AD, verán un banner que les permitirá probar la experiencia actualizada en Mi acceso. Tenga en cuenta que la experiencia de revisiones de acceso actualizada ofrece la misma funcionalidad que la experiencia actual, pero con una interfaz de usuario mejorada sobre nuevas funcionalidades para permitir que los usuarios sean productivos. [Aquí encontrará más información sobre la experiencia actualizada](../governance/perform-access-review.md). Esta versión preliminar pública durará hasta finales de julio de 2020. A finales de julio, los revisores que no hayan participado en la experiencia en versión preliminar serán redirigidos automáticamente a Mi acceso para realizar las revisiones de acceso. Si desea que los revisores cambien permanentemente a la experiencia en versión preliminar en Mi acceso ahora, [realice una solicitud aquí](https://forms.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR5dv-S62099HtxdeKIcgO-NUOFJaRDFDWUpHRk8zQ1BWVU1MMTcyQ1FFUi4u).

---

### <a name="workday-inbound-user-provisioning-and-writeback-apps-now-support-the-latest-versions-of-workday-web-services-api"></a>Las aplicaciones de aprovisionamiento y escritura diferida de usuarios de entrada de WorkDay ahora admiten las versiones más recientes de la API Workday Web Services

**Tipo:** Característica modificada

**Categoría del servicio:** Aprovisionamiento de aplicaciones

**Funcionalidad del producto:** 

Hemos tenido en cuenta los comentarios de los clientes y hemos actualizado las aplicaciones de aprovisionamiento y escritura diferida de usuarios de entrada de WorkDay en la galería de aplicaciones empresariales para admitir las versiones más recientes de la API Workday Web Services (WWS). Con este cambio, los clientes pueden especificar la versión de la API WWS que desean usar en la cadena de conexión. Esto proporciona a los clientes la capacidad de recuperar más atributos de recursos humanos disponibles en las versiones de WorkDay. La aplicación de escritura diferida de WorkDay ahora usa el servicio web de WorkDay recomendado, Change_Work_Contact_Info, para superar las limitaciones de Maintain_Contact_Info.

De forma predeterminada, si no se especifica ninguna versión en la cadena de conexión, las aplicaciones de aprovisionamiento de entrada de WorkDay seguirán usando WWS v21.1. Para cambiar a las versiones más recientes de la API Workday Web Services para el aprovisionamiento de usuarios de entrada, los clientes deben actualizar la cadena de conexión tal y como se documenta [en el tutorial](../saas-apps/workday-inbound-tutorial.md#which-workday-apis-does-the-solution-use-to-query-and-update-workday-worker-profiles), y también actualizar los valores de XPATH utilizados para los atributos de WorkDay, tal y como se describe en [Guía de referencia de atributos de Workday](../app-provisioning/workday-attribute-reference.md#xpath-values-for-workday-web-services-wws-api-v30). 

Para usar la nueva API para la escritura diferida, no se requieren cambios en la aplicación de aprovisionamiento de escritura diferida de WorkDay. En WorkDay, asegúrese de que la cuenta de usuario del sistema de integración (ISU) de WorkDay tiene permisos para invocar el proceso empresarial Change_Work_Contact, tal y como se describe en la sección del tutorial [Configuración de los permisos de la directiva de seguridad de procesos empresariales](../saas-apps/workday-inbound-tutorial.md#configuring-business-process-security-policy-permissions). 

Hemos actualizado la [guía de tutoriales](../saas-apps/workday-inbound-tutorial.md) para reflejar la compatibilidad con la nueva versión de la API.

---

### <a name="users-with-default-access-role-are-now-in-scope-for-provisioning"></a>Los usuarios con el rol de acceso predeterminado ahora están dentro del ámbito del aprovisionamiento

**Tipo:** Característica modificada

**Categoría del servicio:** Aprovisionamiento de aplicaciones

**Funcionalidad del producto:** Administración del ciclo de vida de la identidad

Históricamente, los usuarios con el rol de acceso predeterminado han estado fuera del ámbito del aprovisionamiento. Hemos recibido comentarios acerca de que los clientes quieren que los usuarios con este rol estén dentro del ámbito del aprovisionamiento. A partir del 16 de abril de 2020, todas las nuevas configuraciones de aprovisionamiento permiten aprovisionar a los usuarios con el rol de acceso predeterminado. Cambiaremos gradualmente el comportamiento de las configuraciones de aprovisionamiento existentes para admitir el aprovisionamiento de usuarios con este rol. [Más información.](../app-provisioning/application-provisioning-config-problem-no-users-provisioned.md)

---

### <a name="updated-provisioning-ui"></a>Actualización de la interfaz de usuario de aprovisionamiento

**Tipo:** Característica modificada

**Categoría del servicio:** Aprovisionamiento de aplicaciones

**Funcionalidad del producto:** Administración del ciclo de vida de la identidad

Hemos actualizado la experiencia de aprovisionamiento para crear una vista de administración más centrada. Cuando vaya a la hoja de aprovisionamiento de una aplicación empresarial que ya se ha configurado, podrá supervisar fácilmente el progreso del aprovisionamiento y administrar acciones como el inicio, la detención y el reinicio del aprovisionamiento. [Más información.](../app-provisioning/configure-automatic-user-provisioning-portal.md)

---

### <a name="dynamic-group-rule-validation-is-now-available-for-public-preview"></a>La validación de reglas de grupos dinámicos ahora está disponible en versión preliminar pública

**Tipo:** Característica modificada

**Categoría del servicio:** Administración de grupos

**Funcionalidad del producto:** Colaboración

Azure Active Directory (Azure AD) ahora ofrece los medios para validar las reglas de grupos dinámicos. En la pestaña **Validación de las reglas**, puede validar la regla dinámica con los miembros del grupo de ejemplo, para confirmar que la regla funciona según lo previsto. Al crear o actualizar reglas de grupos dinámicos, los administradores quieren saber si un usuario o un dispositivo será miembro del grupo. Esto ayuda a evaluar si el usuario o el dispositivo cumplen los criterios de la regla y ayuda a solucionar problemas cuando no se espera la pertenencia.

Para más información, consulte [Validación de una regla de pertenencia dinámica a grupos (versión preliminar)](../enterprise-users/groups-dynamic-rule-validation.md).

---

### <a name="identity-secure-score---security-defaults-and-mfa-improvement-action-updates"></a>Puntuación de seguridad de la identidad: valores predeterminados de seguridad y actualizaciones de las acciones de mejora de MFA

**Tipo:** Característica modificada

**Categoría del servicio:** N/D

**Funcionalidad del producto:** Seguridad y protección de la identidad

**Compatibilidad con los valores predeterminados de seguridad para las acciones de mejora de Azure AD:** La Puntuación de seguridad de Microsoft se actualizará con acciones de mejora para admitir [valores predeterminados de seguridad en Azure AD](./concept-fundamentals-security-defaults.md), lo que facilita la protección de la organización con una configuración de seguridad preconfigurada para ataques comunes. Esto afectará a las siguientes acciones de mejora:

- Garantizar que todos los usuarios pueden completar la autenticación multifactor para el acceso seguro
- Requerir MFA para roles administrativos
- Habilitar la directiva para bloquear la autenticación heredada
 
**Actualizaciones de las acciones de mejora de MFA:** Para reflejar la necesidad de las empresas de garantizar la mayor seguridad al aplicar directivas que funcionen con su negocio, la Puntuación de seguridad de Microsoft ha eliminado tres acciones de mejora centradas en la autenticación multifactor y ha agregado otras dos.

Acciones de mejora eliminadas:

- Registrar a todos los usuarios para la autenticación multifactor
- Exigir autenticación multifactor para todos los usuarios
- Requerir MFA para roles con privilegios Azure AD

Acciones de mejora agregadas:

- Garantizar que todos los usuarios pueden completar la autenticación multifactor para el acceso seguro
- Requerir MFA para roles administrativos

Estas nuevas acciones de mejora requerirán el registro de usuarios o administradores para la autenticación multifactor (MFA) en todo el directorio y el establecimiento del conjunto adecuado de directivas que se adapten a las necesidades de la organización. El objetivo principal es tener flexibilidad, asegurando al mismo tiempo que todos los usuarios y administradores puedan autenticarse con varios factores o solicitudes de comprobación de identidad basadas en riesgos. Se puede hacer con varias directivas que apliquen decisiones de ámbito o estableciendo valores predeterminados de seguridad (a partir del 16 de marzo) que permitan a Microsoft decidir cuándo desafiar a los usuarios con MFA. [Más información sobre las novedades en la Puntuación de seguridad de Microsoft](/microsoft-365/security/mtp/microsoft-secure-score#whats-new).

---

## <a name="march-2020"></a>Marzo de 2020

### <a name="unmanaged-azure-active-directory-accounts-in-b2b-update-for-march--2021"></a>Cuentas de Azure Active Directory no administradas en la actualización de B2B de marzo de 2021

**Tipo:** Plan de cambio  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
**A partir del 31 de marzo de 2021**, Microsoft dejará de admitir el canje de invitaciones mediante la creación de cuentas de Azure Active Directory (Azure AD) no administradas e inquilinos para escenarios de colaboración B2B. Como preparación para esto, le recomendamos que opte por la [autenticación de código de acceso de un solo uso por correo electrónico](../external-identities/one-time-passcode.md).

---

### <a name="users-with-the-default-access-role-will-be-in-scope-for-provisioning"></a>Los usuarios con el rol de acceso predeterminado estarán dentro del ámbito del aprovisionamiento.

**Tipo:** Plan de cambio  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
Históricamente, los usuarios con el rol de acceso predeterminado han estado fuera del ámbito del aprovisionamiento. Hemos recibido comentarios acerca de que los clientes quieren que los usuarios con este rol estén dentro del ámbito del aprovisionamiento. Estamos trabajando en la implementación de un cambio para que todas las nuevas configuraciones de aprovisionamiento permitan aprovisionar a los usuarios con el rol de acceso predeterminado. Gradualmente, cambiaremos el comportamiento de las configuraciones de aprovisionamiento existentes para admitir el aprovisionamiento de usuarios con este rol. No se requiere ninguna acción del cliente. Publicaremos una actualización en la [documentación](../app-provisioning/application-provisioning-config-problem-no-users-provisioned.md) una vez que este cambio esté en vigor.

---

### <a name="azure-ad-b2b-collaboration-will-be-available-in-microsoft-azure-operated-by-21vianet-azure-china-21vianet-tenants"></a>La colaboración B2B de Azure AD estará disponible en Microsoft Azure controlado por inquilinos de 21Vianet (Azure China 21Vianet).

**Tipo:** Plan de cambio  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Las funcionalidades de colaboración B2B de Azure AD estarán disponibles en Microsoft Azure controlado por los inquilinos de 21Vianet (Azure China 21Vianet), lo que permite a los usuarios de un inquilino de Azure China 21Vianet colaborar sin problemas con los usuarios de otros inquilinos de Azure China 21Vianet. [Más información sobre la colaboración B2B de Azure AD](/azure/active-directory/b2b/).

---
 
### <a name="azure-ad-b2b-collaboration-invitation-email-redesign"></a>Rediseño del correo electrónico de invitación de colaboración B2B de Azure AD

**Tipo:** Plan de cambio  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Los [correos electrónicos](../external-identities/invitation-email-elements.md) enviados por el servicio de invitación de colaboración B2B de Azure AD para invitar a los usuarios al directorio se rediseñarán para que la información de la invitación y los siguientes pasos del usuario sean más claros.

---

### <a name="homerealmdiscovery-policy-changes-will-appear-in-the-audit-logs"></a>Los cambios de la directiva HomeRealmDiscovery aparecerán en los registros de auditoría

**Tipo:** Corregido  
**Categoría del servicio:** Auditoría  
**Funcionalidad del producto:** Supervisión e informes
 
Se ha corregido un error por el que los cambios en la [directiva HomeRealmDiscovery](../manage-apps/configure-authentication-for-federated-users-portal.md) no se incluían en los registros de auditoría. Ahora podrá ver cuándo y cómo ha cambiado la directiva, y quién la cambió. 

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---march-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (marzo de 2020)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En marzo de 2020, hemos agregado estas 51 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones: 

[Cisco AnyConnect](../saas-apps/cisco-anyconnect.md), [Zoho One China](../saas-apps/zoho-one-china-tutorial.md), [PlusPlus](https://test.plusplus.app/auth/login/azuread-outlook/), [Profit.co SAML App](../saas-apps/profitco-saml-app-tutorial.md), [iPoint Service Provider](../saas-apps/ipoint-service-provider-tutorial.md), [contexxt.ai SPHERE](https://contexxt-sphere.com/login), [Wisdom By Invictus](../saas-apps/wisdom-by-invictus-tutorial.md), [Flare Digital Signage](https://spark-dev.pixelnebula.com/login), [Logz.io - Cloud Observability for Engineers](../saas-apps/logzio-cloud-observability-for-engineers-tutorial.md), [SpectrumU](../saas-apps/spectrumu-tutorial.md), [BizzContact](https://www.bizzcontact.app/), [Elqano SSO](../saas-apps/elqano-sso-tutorial.md), [MarketSignShare](http://www.signshare.com/), [CrossKnowledge Learning Suite](../saas-apps/crossknowledge-learning-suite-tutorial.md), [Netvision Compas](../saas-apps/netvision-compas-tutorial.md), [FCM HUB](../saas-apps/fcm-hub-tutorial.md), [RIB A/S Byggeweb Mobile](https://apps.apple.com/us/app/docia/id529058757), [GoLinks](../saas-apps/golinks-tutorial.md), [Datadog](../saas-apps/datadog-tutorial.md), [Zscaler B2B User Portal](../saas-apps/zscaler-b2b-user-portal-tutorial.md), [LIFT](../saas-apps/lift-tutorial.md), [Planview Enterprise One](../saas-apps/planview-enterprise-one-tutorial.md), [WatchTeams](https://www.devfinition.com/), [Aster](https://demo.asterapp.io/login), [Skills Workflow](../saas-apps/skills-workflow-tutorial.md), [Node Insight](https://admin.nodeinsight.com/AADLogin.aspx), [IP Platform](../saas-apps/ip-platform-tutorial.md), [InVision](../saas-apps/invision-tutorial.md), [Pipedrive](../saas-apps/pipedrive-tutorial.md), [Showcase Workshop](https://app.showcaseworkshop.com/), [Greenlight Integration Platform](../saas-apps/greenlight-integration-platform-tutorial.md), [Greenlight Compliant Access Management](../saas-apps/greenlight-compliant-access-management-tutorial.md), [Grok Learning](../saas-apps/grok-learning-tutorial.md), [Miradore Online](https://login.online.miradore.com/), [Khoros Care](../saas-apps/khoros-care-tutorial.md), [AskYourTeam](../saas-apps/askyourteam-tutorial.md), [TruNarrative](../saas-apps/trunarrative-tutorial.md), [Smartwaiver](https://www.smartwaiver.com/m/user/sw_login.php?wms_login), [Bizagi Studio for Digital Process Automation](../saas-apps/bizagi-studio-for-digital-process-automation-tutorial.md), [insuiteX](https://www.insuite.jp/), [sybo](https://www.systexsoftware.com.tw/), [Britive](../saas-apps/britive-tutorial.md), [WhosOffice](../saas-apps/whosoffice-tutorial.md), [E-days](../saas-apps/e-days-tutorial.md), [Kollective SDN](https://portal.kollective.app/login), [Witivio](https://app.witivio.com/), [Playvox](https://my.playvox.com/login), [Korn Ferry 360](../saas-apps/korn-ferry-360-tutorial.md), [Campus Café](../saas-apps/campus-cafe-tutorial.md), [Catchpoint](../saas-apps/catchpoint-tutorial.md), [Code42](../saas-apps/code42-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="azure-ad-b2b-collaboration-available-in-azure-government-tenants"></a>Está disponible la colaboración B2B de Azure AD en inquilinos de Azure Government

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** B2B/B2C
 
Las características de colaboración B2B de Azure AD ahora están disponibles entre algunos inquilinos de Azure Government.  Para averiguar si su inquilino es capaz de usar estas funcionalidades, siga las instrucciones que se indican en [¿Cómo puedo saber si la colaboración B2B está disponible en mi inquilino de Azure US Government?](../external-identities/current-limitations.md#how-can-i-tell-if-b2b-collaboration-is-available-in-my-azure-us-government-tenant)

---

### <a name="azure-monitor-integration-for-azure-logs-is-now-available-in-azure-government"></a>La integración de Azure Monitor para los registros de Azure ahora está disponible en Azure Government

**Tipo:** Nueva característica  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 
La integración de Azure Monitor en los registros de Azure AD ahora está disponible en Azure Government. Puede enrutar los registros de Azure AD (registros de auditoría e inicio de sesión) a una cuenta de almacenamiento, un centro de eventos y Log Analytics. Consulte la [documentación detallada](../reports-monitoring/concept-activity-logs-azure-monitor.md), así como los [planes de implementación para la elaboración de informes y la supervisión](../reports-monitoring/plan-monitoring-and-reporting.md) para escenarios de Azure AD.

---

### <a name="identity-protection-refresh-in-azure-government"></a>Actualización de Identity Protection en Azure Government

**Tipo:** Nueva característica  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Nos complace compartir que ahora hemos implementado la experiencia actualizada de [Azure AD Identity Protection](../identity-protection/overview-identity-protection.md) en el [portal de Microsoft Azure Government](https://portal.azure.us/). Para obtener más información, consulte nuestra [entrada de blog de anuncio](https://techcommunity.microsoft.com/t5/public-sector-blog/identity-protection-refresh-in-microsoft-azure-government/ba-p/1223667).

---

### <a name="disaster-recovery-download-and-store-your-provisioning-configuration"></a>Recuperación ante desastres: Descarga y almacenamiento de la configuración de aprovisionamiento

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad
 
El servicio de aprovisionamiento de Azure AD proporciona un amplio conjunto de funcionalidades de configuración. Los clientes deben poder guardar su configuración para poder consultarla más adelante o revertir a una versión válida conocida. Hemos agregado la capacidad de descargar la configuración de aprovisionamiento como archivo JSON y cargarlo cuando sea necesario. [Más información](../app-provisioning/export-import-provisioning-configuration.md).

---
 
### <a name="sspr-self-service-password-reset-now-requires-two-gates-for-admins-in-microsoft-azure-operated-by-21vianet-azure-china-21vianet"></a>SSPR (autoservicio de restablecimiento de contraseña) ahora requiere dos puertas para los administradores en Microsoft Azure controlado por 21Vianet (Azure China 21Vianet) 

**Tipo:** Característica modificada  
**Categoría del servicio:** Restablecimiento de contraseña de autoservicio  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Anteriormente, en Microsoft Azure controlado por 21Vianet (Azure China 21Vianet), los administradores que usaban el autoservicio de restablecimiento de contraseña (SSPR) para restablecer sus propias contraseñas solo necesitaban una "puerta" (desafío) para demostrar su identidad. En las nubes públicas y en otras nubes nacionales, los administradores generalmente deben usar dos puertas para demostrar su identidad al usar SSPR. No obstante, dado que no se admiten las llamadas telefónicas ni SMS en Azure China 21Vianet, hemos permitido que los administradores restablezcan la contraseña con una sola puerta.

Estamos creando la paridad de características de SSPR entre Azure China 21Vianet y la nube pública. En el futuro, los administradores deberán usar dos puertas al usar SSPR. Se admitirán SMS, llamadas telefónicas, y códigos y notificaciones de aplicación de autenticador. [Más información](../authentication/concept-sspr-policy.md#administrator-reset-policy-differences).

---

### <a name="password-length-is-limited-to-256-characters"></a>La longitud de las contraseñas está limitada a 256 caracteres

**Tipo:** Característica modificada  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
Para garantizar la confiabilidad del servicio de Azure AD, las contraseñas de usuario ahora tienen una longitud máxima de 256 caracteres. A los usuarios con contraseñas más largas que esta se les pedirá que cambien su contraseña en el inicio de sesión subsiguiente, ya sea poniéndose en contacto con su administrador o mediante la característica de autoservicio de restablecimiento de contraseña.

Este cambio se habilitó el 13 de marzo de 2020, a las 10:00 PST (18:00 UTC), y el error es AADSTS 50052, InvalidPasswordExceedsMaxLength. Para obtener más información, consulte el [aviso de cambio importante](../develop/reference-breaking-changes.md#user-passwords-will-be-restricted-to-256-characters).

---

### <a name="azure-ad-sign-in-logs-are-now-available-for-all-free-tenants-through-the-azure-portal"></a>Los registros de inicio de sesión de Azure AD ahora están disponibles para todos los inquilinos gratuitos a través de Azure Portal

**Tipo:** Característica modificada  
**Categoría del servicio:** Notificación  
**Funcionalidad del producto:** Supervisión e informes
 
A partir de ahora, los clientes que dispongan de inquilinos gratuitos pueden tener acceso a los [registros de inicio de sesión de Azure AD desde Azure Portal](../reports-monitoring/concept-sign-ins.md) para un máximo de 7 días. Anteriormente, los registros de inicio de sesión solo estaban disponibles para los clientes con licencias de Azure Active Directory Premium. Con este cambio, todos los inquilinos pueden tener acceso a estos registros a través del portal.

> [!NOTE]
> Los clientes todavía necesitan una licencia Premium (Azure Active Directory Premium P1 o P2) para tener acceso a los registros de inicio de sesión a través de Microsoft Graph API y Azure Monitor.

---

### <a name="deprecation-of-directory-wide-groups-option-from-groups-general-settings-on-azure-portal"></a>Opción en desuso de grupos de todo el directorio de la configuración general de grupos en Azure Portal

**Tipo:** Obsoleto  
**Categoría del servicio:** Administración de grupos  
**Funcionalidad del producto:** Colaboración

Para ofrecer a los clientes una manera más flexible de crear grupos de todo el directorio que se adapte mejor a sus necesidades, hemos reemplazado la opción **Grupos de todo el directorio** de la opción **Grupos** > **General** en Azure Portal por un vínculo a la [documentación de grupos dinámicos](../enterprise-users/groups-dynamic-membership.md). Hemos mejorado la documentación para incluir más instrucciones para que los administradores puedan crear grupos de todos los usuarios que incluyan o excluyan usuarios invitados.

---

## <a name="february-2020"></a>Febrero de 2020

### <a name="upcoming-changes-to-custom-controls"></a>Próximos cambios en los controles personalizados

**Tipo:** Plan de cambio  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Tenemos previsto reemplazar la versión preliminar actual de los controles personalizados por un enfoque que permita que las funcionalidades de autenticación proporcionadas por el partner funcionen sin problemas con las experiencias de administrador y usuario final de Azure Active Directory. Hoy en día, las soluciones de MFA de los partners enfrentan las siguientes limitaciones: solo funcionan después de haber escrito una contraseña; no sirven como MFA para la autenticación de nivel superior en otros escenarios clave; y no se integran con las funciones de administración de credenciales administrativas ni de usuario final. La nueva implementación permitirá que los factores de autenticación proporcionados por los partners funcionen junto con factores integrados para escenarios clave, como el registro, el uso, las notificaciones de MFA, la autenticación de nivel superior, la creación de informes y el registro. 

Los controles personalizados se seguirán admitiendo en la versión preliminar junto con el nuevo diseño hasta que alcancen la disponibilidad general. En ese momento, daremos tiempo a los clientes para migrar al nuevo diseño. Debido a las limitaciones del enfoque actual, no se incorporarán nuevos proveedores hasta que el nuevo diseño esté disponible. Estamos trabajando en estrecha colaboración con los clientes y proveedores, y comunicaremos los plazos a medida que nos acerquemos. [Más información](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/upcoming-changes-to-custom-controls/ba-p/1144696#).

---

### <a name="identity-secure-score---mfa-improvement-action-updates"></a>Puntuación de seguridad de la identidad: actualizaciones de acciones de mejora de MFA

**Tipo:** Plan de cambio  
**Categoría del servicio:** MFA  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Para reflejar la necesidad de las empresas de garantizar la mayor seguridad al aplicar directivas que funcionen con su negocio, la puntuación segura de Microsoft va a quitar tres acciones de mejora centradas en la autenticación multifactor (MFA) y va a añadir otras dos.

Se quitarán las siguientes acciones de mejora:

- Registrar todos los usuarios de MFA
- Exigir autenticación multifactor para todos los usuarios
- Requerir MFA para roles con privilegios Azure AD

Se agregarán las siguientes acciones de mejora:

- Asegurarse de que todos los usuarios pueden completar MFA para el acceso seguro
- Requerir MFA para roles administrativos

Estas nuevas acciones de mejora requerirán el registro de usuarios o administradores para MFA en todo el directorio y el establecimiento del conjunto adecuado de directivas que se adapten a las necesidades de la organización. El objetivo principal es tener flexibilidad, asegurando al mismo tiempo que todos los usuarios y administradores puedan autenticarse con varios factores o solicitudes de comprobación de identidad basadas en riesgos. Esto puede traducirse en la configuración de valores predeterminados de seguridad que permitan a Microsoft decidir cuándo desafiar a los usuarios para MFA o el mantenimiento de varias directivas que apliquen decisiones de ámbito. Como parte de estas actualizaciones de acciones de mejora, las directivas de protección de línea de base ya no se incluirán en los cálculos de puntuación. [Obtenga más información sobre lo próximo de la puntuación segura de Microsoft](/microsoft-365/security/mtp/microsoft-secure-score-whats-coming).

---

### <a name="azure-ad-domain-services-sku-selection"></a>Selección de SKU Azure AD Domain Services

**Tipo:** Nueva característica  
**Categoría del servicio:** Azure AD Domain Services  
**Funcionalidad del producto:** Azure AD Domain Services
 
Hemos escuchado los comentarios de los clientes de Azure AD Domain Services que desean más flexibilidad a la hora de seleccionar los niveles de rendimiento de sus instancias. A partir del 1 de febrero de 2020, cambiamos de un modelo dinámico (donde Azure AD determina el rendimiento y el plan de tarifa en función del número de objetos) a un modelo de selección automática. Ahora los clientes pueden elegir un nivel de rendimiento que se adapte a su entorno. Este cambio también nos permite habilitar nuevos escenarios, como los bosques de recursos, y características Premium, como las copias de seguridad diarias. El número de objetos es ahora ilimitado para todas las SKU, pero seguiremos ofreciendo sugerencias de cantidades de objetos para cada nivel.

**No se requiere ninguna acción inmediata del cliente.** Para los clientes actuales, el nivel dinámico que estaba en uso el 1 de febrero de 2020 determina el nuevo nivel predeterminado. El resultado de este cambio no afecta al precio ni al rendimiento. A partir de entonces, los clientes de Azure AD DS deberán evaluar los requisitos de rendimiento a medida que cambien sus características de carga de trabajo y el tamaño del directorio. El cambio entre los niveles de servicio seguirá siendo una operación sin tiempo de inactividad, y ya no se moverá automáticamente a los clientes a nuevos niveles en función del crecimiento de su directorio. Además, no habrá ningún aumento del precio y los nuevos precios se alinearán con nuestro modelo de facturación actual. Para obtener más información, consulte la [documentación de las SKU de Azure AD DS](../../active-directory-domain-services/administration-concepts.md#azure-ad-ds-skus) y la [página de precios de Azure AD Domain Services](https://azure.microsoft.com/pricing/details/active-directory-ds/).

---
 
### <a name="new-federated-apps-available-in-azure-ad-app-gallery---february-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (febrero de 2020)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En febrero de 2020, hemos agregado estas 31 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones: 

[IamIP Patent Platform](../saas-apps/iamip-patent-platform-tutorial.md), [Experience Cloud](../saas-apps/experience-cloud-tutorial.md), [NS1 SSO For Azure](../saas-apps/ns1-sso-azure-tutorial.md), [Barracuda Email Security Service](https://ess.barracudanetworks.com/sso/azure), [ABa Reporting](https://myaba.co.uk/client-access/signin/auth/msad), [In Case of Crisis - Online Portal](../saas-apps/in-case-of-crisis-online-portal-tutorial.md), [BIC Cloud Design](../saas-apps/bic-cloud-design-tutorial.md), [Beekeeper Azure AD Data Connector](../saas-apps/beekeeper-azure-ad-data-connector-tutorial.md), [Korn Ferry Assessments](https://www.kornferry.com/solutions/kf-digital/kf-assess), [Verkada Command](../saas-apps/verkada-command-tutorial.md), [Splashtop](../saas-apps/splashtop-tutorial.md), [Syxsense](../saas-apps/syxsense-tutorial.md), [EAB Navigate](../saas-apps/eab-navigate-tutorial.md), [New Relic (Limited Release)](../saas-apps/new-relic-limited-release-tutorial.md), [Thulium](https://admin.thulium.com/login/instance), [Ticket Manager](../saas-apps/ticketmanager-tutorial.md), [Template Chooser for Teams](https://links.officeatwork.com/templatechooser-download-teams), [Beesy](https://www.beesy.me/index.php/site/login), [Health Support System](../saas-apps/health-support-system-tutorial.md), [MURAL](https://app.mural.co/signup), [Hive](../saas-apps/hive-tutorial.md), [LavaDo](https://appsource.microsoft.com/product/web-apps/lavaloon.lavado_standard?tab=Overview), [Wakelet](https://wakelet.com/login), [Firmex VDR](../saas-apps/firmex-vdr-tutorial.md), [ThingLink for Teachers and Schools](https://www.thinglink.com/), [Coda](../saas-apps/coda-tutorial.md), [NearpodApp](https://nearpod.com/signup/?oc=Microsoft&utm_campaign=Microsoft&utm_medium=site&utm_source=product), [WEDO](../saas-apps/wedo-tutorial.md), [InvitePeople](https://invitepeople.com/login), [Reprints Desk - Article Galaxy](../saas-apps/reprints-desk-article-galaxy-tutorial.md), [TeamViewer](../saas-apps/teamviewer-tutorial.md)

 
Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---
 
### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---february-2020"></a>Nuevos conectores de aprovisionamiento en la galería de aplicaciones de Azure AD (febrero de 2020)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Mixpanel](../saas-apps/mixpanel-provisioning-tutorial.md)
- [TeamViewer](../saas-apps/teamviewer-provisioning-tutorial.md)
- [Azure Databricks](/azure/databricks/administration-guide/users-groups/scim/aad)
- [PureCloud by Genesys](../saas-apps/purecloud-by-genesys-provisioning-tutorial.md)
- [Zapier](../saas-apps/zapier-provisioning-tutorial.md)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

---
 
### <a name="azure-ad-support-for-fido2-security-keys-in-hybrid-environments"></a>Compatibilidad de Azure AD con las claves de seguridad de FIDO2 en entornos híbridos

**Tipo:** Nueva característica  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
Anunciamos la versión preliminar pública de la compatibilidad de Azure AD con las claves de seguridad de FIDO2 en entornos híbridos. Los usuarios ahora pueden usar las claves de seguridad de FIDO2 para iniciar sesión en sus dispositivos de Windows 10 unidos a Azure AD híbrido y obtener un inicio de sesión fluido en sus recursos locales y en la nube. La compatibilidad con entornos híbridos ha sido la característica más solicitada de nuestros clientes sin contraseña, ya que inicialmente lanzamos la versión preliminar pública para la compatibilidad con FIDO2 en dispositivos unidos a Azure AD. La autenticación sin contraseña mediante tecnologías avanzadas como la biométrica y la criptografía de clave privada y pública es cómoda y fácil de usar a la vez que segura. Con esta versión preliminar pública, ahora puede usar la autenticación moderna como las claves de seguridad de FIDO2 para acceder a los recursos de Active Directory tradicionales. Para obtener más información, vaya a [Inicio de sesión único en recursos locales](../authentication/howto-authentication-passwordless-security-key-on-premises.md). 

Para empezar, visite [Habilitación de las claves de seguridad FIDO2 para un inquilino](../authentication/howto-authentication-passwordless-security-key.md) para obtener instrucciones paso a paso. 

---
 
### <a name="the-new-my-account-experience-is-now-generally-available"></a>La nueva experiencia de Mi cuenta ya está disponible con carácter general.

**Tipo:** Característica modificada  
**Categoría del servicio:** Mi perfil/cuenta  
**Funcionalidad del producto:** Experiencias de usuario final
 
Mi cuenta, el punto en el que se atienden todas las necesidades de administración de las cuentas de usuario final, ya está disponible con carácter general. Los usuarios finales pueden tener acceso a este nuevo sitio a través de la dirección URL o en el encabezado de la nueva experiencia Mis aplicaciones. Obtenga más información sobre todas las funcionalidades de autoservicio que ofrece la nueva experiencia en [¿Qué es el portal Mi cuenta?](../user-help/my-account-portal-overview.md)

---
 
### <a name="my-account-site-url-updating-to-myaccountmicrosoftcom"></a>Actualización de la dirección URL del sitio de Mi cuenta a myaccount.microsoft.com

**Tipo:** Característica modificada  
**Categoría del servicio:** Mi perfil/cuenta  
**Funcionalidad del producto:** Experiencias de usuario final
 
La nueva experiencia de usuario final de Mi cuenta actualizará su dirección URL a `https://myaccount.microsoft.com` el próximo mes. Obtenga más información sobre la experiencia y todas las funcionalidades de autoservicio de cuenta que ofrece a los usuarios finales en la [ayuda del portal Mi cuenta](../user-help/my-account-portal-overview.md).

---

## <a name="january-2020"></a>Enero de 2020
 
### <a name="the-new-my-apps-portal-is-now-generally-available"></a>El nuevo portal Mis aplicaciones ya está disponible con carácter general

**Tipo:** Plan de cambio  
**Categoría del servicio:** Mis aplicaciones  
**Funcionalidad del producto:** Experiencias de usuario final
 
Actualice su organización al nuevo portal Mis aplicaciones que ya está disponible con carácter general. Puede encontrar más información sobre el nuevo portal y las colecciones en [Creación de colecciones en el portal Mis aplicaciones](../manage-apps/access-panel-collections.md).

---
 
### <a name="workspaces-in-azure-ad-have-been-renamed-to-collections"></a>Las áreas de trabajo de Azure AD se han renombrado a colecciones

**Tipo:** Característica modificada  
**Categoría del servicio:** Mis aplicaciones   
**Funcionalidad del producto:** Experiencias de usuario final
 
Las áreas de trabajo, que son filtros que los administradores pueden configurar para organizar las aplicaciones de sus usuarios, ahora se llamarán colecciones. Obtenga más información sobre cómo configurarlas en [Creación de colecciones en el portal Mis aplicaciones](../manage-apps/access-panel-collections.md).

---
 
### <a name="azure-ad-b2c-phone-sign-up-and-sign-in-using-custom-policy-public-preview"></a>Registro e inicio de sesión mediante teléfono en Azure AD B2C con una directiva personalizada (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C
 
Con el registro y el inicio de sesión mediante el número de teléfono, los desarrolladores y las empresas pueden permitir a sus clientes registrarse e iniciar sesión con una contraseña de un solo uso enviada al número de teléfono del usuario a través de un SMS. Esta característica también permite al cliente cambiar su número de teléfono si pierde el acceso a su dispositivo. Con la eficacia de las directivas personalizadas y el registro e inicio de sesión mediante el teléfono, los desarrolladores y las empresas pueden comunicar su marca a través de la personalización de la página. Aprenda cómo [configurar el registro y el inicio de sesión en el teléfono con directivas personalizadas en Azure AD B2C](../../active-directory-b2c/phone-authentication-user-flows.md).
 
---
 
### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---january-2020"></a>Nuevos conectores de aprovisionamiento en la galería de aplicaciones de Azure AD (enero de 2020)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Promapp]( ../saas-apps/promapp-provisioning-tutorial.md)
- [Zscaler Private Access](../saas-apps/zscaler-private-access-provisioning-tutorial.md)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

---
 
### <a name="new-federated-apps-available-in-azure-ad-app-gallery---january-2020"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (enero de 2020)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros
 
En enero de 2020, hemos agregado estas 33 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones: 

[JOSA](../saas-apps/josa-tutorial.md), [Fastly Edge Cloud](../saas-apps/fastly-edge-cloud-tutorial.md), [Terraform Enterprise](../saas-apps/terraform-enterprise-tutorial.md), [Spintr SSO](../saas-apps/spintr-sso-tutorial.md), [Abibot Netlogistik](https://azuremarketplace.microsoft.com/marketplace/apps/aad.abibotnetlogistik), [SkyKick](https://login.skykick.com/login?state=g6Fo2SBTd3M5Q0xBT0JMd3luS2JUTGlYN3pYTE1remJQZnR1c6N0aWTZIDhCSkwzYVQxX2ZMZjNUaWxNUHhCSXg2OHJzbllTcmYto2NpZNkgM0h6czk3ZlF6aFNJV1VNVWQzMmpHeFFDbDRIMkx5VEc&client=3Hzs97fQzhSIWUMUd32jGxQCl4H2LyTG&protocol=oauth2&audience=https://papi.skykick.com&response_type=code&redirect_uri=https://portal.skykick.com/callback&scope=openid%20profile%20offline_access), [Upshotly](../saas-apps/upshotly-tutorial.md), [LeaveBot](https://appsource.microsoft.com/en-us/product/office/WA200001175), [DataCamp](../saas-apps/datacamp-tutorial.md), [TripActions](../saas-apps/tripactions-tutorial.md), [SmartWork](https://www.intumit.com/teams-smartwork/), [Dotcom-Monitor](../saas-apps/dotcom-monitor-tutorial.md), [SSOGEN - Azure AD SSO Gateway for Oracle E-Business Suite - EBS, PeopleSoft y JDE](../saas-apps/ssogen-tutorial.md), [Hosted MyCirqa SSO](../saas-apps/hosted-mycirqa-sso-tutorial.md), [Yuhu Property Management Platform](../saas-apps/yuhu-property-management-platform-tutorial.md), [LumApps](https://sites.lumapps.com/login), [Upwork Enterprise](../saas-apps/upwork-enterprise-tutorial.md), [Talentsoft](../saas-apps/talentsoft-tutorial.md), [SmartDB for Microsoft Teams](http://teams.smartdb.jp/login/), [PressPage](../saas-apps/presspage-tutorial.md), [ContractSafe Saml2 SSO](../saas-apps/contractsafe-saml2-sso-tutorial.md), [Maxient Conduct Manager Software](../saas-apps/maxient-conduct-manager-software-tutorial.md), [Helpshift](../saas-apps/helpshift-tutorial.md), [PortalTalk 365](https://www.portaltalk.com/), [CoreView](https://portal.coreview.com/), Squelch Cloud Office365 Connector, [PingFlow Authentication](https://app-staging.pingview.io/), [ PrinterLogic SaaS](../saas-apps/printerlogic-saas-tutorial.md), [Taskize Connect](../saas-apps/taskize-connect-tutorial.md), [Sandwai](https://app.sandwai.com/), [EZRentOut](../saas-apps/ezrentout-tutorial.md), [AssetSonar](../saas-apps/assetsonar-tutorial.md), [Akari Virtual Assistant](https://akari.io/akari-virtual-assistant/)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="two-new-identity-protection-detections"></a>Dos nuevas detecciones de Identity Protection

**Tipo:** Nueva característica  
**Categoría del servicio:** Protección de identidad  
**Funcionalidad del producto:** Seguridad y protección de la identidad
 
Hemos agregado a Identity Protection dos nuevos tipos de detección vinculados al inicio de sesión: Reglas de manipulación sospechosa de la bandeja de entrada y Viaje imposible. Microsoft Cloud App Security (MCAS) descubre estas detecciones sin conexión e influye en los riesgos de usuario y de inicio de sesión en Identity Protection. Para más información sobre estas detecciones, consulte nuestros [tipos de riesgo de inicio de sesión](../identity-protection/concept-identity-protection-risks.md#sign-in-risk).
 
---
 
### <a name="breaking-change-uri-fragments-will-not-be-carried-through-the-login-redirect"></a>Cambio de última hora: los fragmentos de URI no se trasladarán a través de la redirección de inicio de sesión

**Tipo:** Característica modificada  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios
 
A partir del 8 de febrero de 2020, cuando se envíe una solicitud a login.microsoftonline.com para que un usuario inicie sesión, el servicio anexará un fragmento vacío a la solicitud.  Esto evita una clase de ataques de redireccionamiento, ya que se asegura de que el explorador borra todo fragmento existente en la solicitud. Ninguna aplicación debe tener una dependencia de este comportamiento. Para más información, consulte [Cambios importantes](../develop/reference-breaking-changes.md#february-2020) en la documentación de la plataforma de identidad de Microsoft.

---

## <a name="december-2019"></a>Diciembre de 2019

### <a name="integrate-sap-successfactors-provisioning-into-azure-ad-and-on-premises-ad-public-preview"></a>Integración del aprovisionamiento de SAP SuccessFactors en Azure AD y AD local (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aprovisionamiento de aplicaciones  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad

Ahora puede integrar SAP SuccessFactors como origen de identidad relevante en Azure AD. Esta integración le ayuda a automatizar el ciclo de vida de la identidad de un extremo a otro, incluido el uso de eventos basados en recursos humanos, como nuevas contrataciones o resoluciones de contrato, para controlar el aprovisionamiento de cuentas de Azure AD.

Para obtener más información sobre cómo configurar el aprovisionamiento de entrada de SAP SuccessFactors para Azure AD, consulte el tutorial sobre [configuración del aprovisionamiento automático de SAP SuccessFactors](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md).

---

### <a name="support-for-customized-emails-in-azure-ad-b2c-public-preview"></a>Compatibilidad con correos electrónicos personalizados en Azure AD B2C (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** B2C: administración de identidades de consumidor  
**Funcionalidad del producto:** B2B/B2C

Ahora puede usar Azure AD B2C para crear correos electrónicos personalizados cuando los usuarios se suscriban para usar las aplicaciones. Mediante el uso de DisplayControls (actualmente en versión preliminar) y un proveedor de correo electrónico de terceros (por ejemplo, [SendGrid](https://sendgrid.com/), [SparkPost](https://sparkpost.com/) o una API REST personalizada), puede usar su propia plantilla de correo electrónico, dirección **De** y texto del asunto, además de admitir la localización y la configuración personalizada de contraseña de un solo uso (OTP).

Para obtener más información, consulte el artículo sobre [comprobación de correo electrónico personalizada en Azure Active Directory B2C](../../active-directory-b2c/custom-email-sendgrid.md).

---

### <a name="replacement-of-baseline-policies-with-security-defaults"></a>Sustitución de directivas de base de referencia por valores predeterminados de seguridad

**Tipo:** Característica modificada  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Como parte de un modelo predeterminado seguro para la autenticación, vamos a quitar las directivas de protección de base de referencia existentes de todos los inquilinos. Se prevé que esta retirada se complete a finales de febrero. El reemplazo de estas directivas de protección de base de referencia son los valores predeterminados de seguridad. Si ha usado directivas de protección de base de referencia, debe planear la migración a la nueva directiva de valores predeterminados de seguridad o al acceso condicional. Si no ha usado estas directivas, no tiene que realizar ninguna acción.

Para obtener más información sobre los nuevos valores predeterminados de seguridad, consulte [¿Qué son los valores predeterminados de seguridad?](./concept-fundamentals-security-defaults.md) Para obtener más información sobre las directivas de acceso condicional, consulte [Directivas de acceso condicional habituales](../conditional-access/concept-conditional-access-policy-common.md).

---

## <a name="november-2019"></a>Noviembre de 2019

### <a name="support-for-the-samesite-attribute-and-chrome-80"></a>Compatibilidad con el atributo SameSite y Chrome 80

**Tipo:** Plan de cambio  
**Categoría del servicio:** Autenticaciones (inicios de sesión)  
**Funcionalidad del producto:** Autenticación de usuarios

Como parte de un modelo predeterminado y seguro para las cookies, el explorador Chrome 80 cambia el modo en que trata las cookies sin el atributo `SameSite`. Cualquier cookie que no especifique el atributo `SameSite` se tratará como si se hubiera establecido en `SameSite=Lax`, lo que producirá un bloqueo de Chrome en ciertos escenarios de uso compartido de cookies entre dominios de los que puede depender su aplicación. Para mantener el comportamiento de Chrome anterior, puede utilizar el atributo `SameSite=None` y agregar un atributo `Secure` adicional, por lo que solo se puede tener acceso a las cookies entre sitios a través de conexiones HTTPS. Chrome está programado para completar este cambio el 4 de febrero de 2020.

Se recomienda que todos los desarrolladores prueben sus aplicaciones siguiendo esta guía:

- Establecer el valor predeterminado de la opción **Usar cookies seguras** en **Sí**.

- Establecer el valor predeterminado del atributo **SameSite** en **Ninguno**.

- Agregar un atributo `SameSite` adicional de **Seguro**.

Para obtener más información, consulte el artículo sobre [próximos cambios en la cookie de SameSite en ASP.NET y ASP.NET Core](https://devblogs.microsoft.com/aspnet/upcoming-samesite-cookie-changes-in-asp-net-and-asp-net-core/) y [Posible interrupción para los sitios web de los clientes y los servicios de Microsoft en la versión 79 de Chrome Beta y versiones posteriores](https://support.microsoft.com/help/4522904/potential-disruption-to-microsoft-services-in-chrome-beta-version-79).

---

### <a name="new-hotfix-for-microsoft-identity-manager-mim-2016-service-pack-2-sp2"></a>Nueva revisión para Microsoft Identity Manager (MIM) 2016 Service Pack 2 (SP2)

**Tipo:** Corregido  
**Categoría del servicio:** Microsoft Identity Manager  
**Funcionalidad del producto:** Administración del ciclo de vida de la identidad

Un paquete acumulativo de revisiones (compilación 4.6.34.0) está disponible para Microsoft Identity Manager (MIM) 2016 Service Pack 2 (SP2). Este paquete acumulativo resuelve problemas y agrega mejoras que se describen en la sección sobre "problemas corregidos y mejoras agregadas en esta actualización".

Para obtener más información y descargar el paquete de revisiones, consulte el artículo [El paquete acumulativo de actualizaciones de Microsoft Identity Manager 2016 Service Pack 2 (compilación 4.6.34.0) está disponible](https://support.microsoft.com/help/4512924/microsoft-identity-manager-2016-service-pack-2-build-4-6-34-0-update-r).

---

### <a name="new-ad-fs-app-activity-report-to-help-migrate-apps-to-azure-ad-public-preview"></a>Nuevo informe de actividad de aplicaciones de AD FS para ayudar a migrar las aplicaciones a Azure AD (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO

Use el nuevo informe de actividad de aplicaciones de Servicios de federación de Active Directory (AD FS) en Azure Portal para identificar las aplicaciones que se pueden migrar a Azure AD. El informe evalúa todas las aplicaciones de AD FS con respecto a la compatibilidad con Azure AD, comprueba si hay problemas y proporciona instrucciones sobre cómo preparar aplicaciones concretas para su migración.

Para obtener más información, consulte [Uso del informe de actividades de aplicaciones de AD FS para migrar aplicaciones a Azure AD](../manage-apps/migrate-adfs-application-activity.md).

---

### <a name="new-workflow-for-users-to-request-administrator-consent-public-preview"></a>Nuevo flujo de trabajo para que los usuarios soliciten el consentimiento del administrador (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Control de acceso

El flujo de trabajo de consentimiento del administrador proporciona a los administradores una manera de conceder acceso a las aplicaciones que requieren la aprobación del administrador. Si un usuario intenta acceder a una aplicación, pero no puede dar su consentimiento, puede enviar una solicitud de aprobación del administrador. La solicitud se envía por correo electrónico, y se coloca en una cola a la que se puede acceder desde Azure Portal, a todos los administradores que se han designado como revisores. Una vez que un revisor realiza una acción en una solicitud pendiente, se notifica la acción a los usuarios que realizan la solicitud.

Para obtener más información, consulte [Configuración del flujo de trabajo de consentimiento del administrador (versión preliminar)](../manage-apps/configure-admin-consent-workflow.md).

---

### <a name="new-azure-ad-app-registrations-token-configuration-experience-for-managing-optional-claims-public-preview"></a>Nueva experiencia de configuración del token de registros de aplicación de Azure AD para administrar notificaciones opcionales (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Experiencia para el desarrollador

La nueva hoja de **configuración del token de registros de aplicación de Azure AD** de Azure Portal muestra ahora a los desarrolladores de aplicaciones una lista dinámica de notificaciones opcionales para sus aplicaciones. Esta nueva experiencia ayuda a simplificar las migraciones de aplicaciones de Azure AD y a minimizar las configuraciones incorrectas de las notificaciones opcionales.

Para obtener más información, consulte [Proporcionar notificaciones opcionales a la aplicación de Azure AD](../develop/active-directory-optional-claims.md).

---

### <a name="new-two-stage-approval-workflow-in-azure-ad-entitlement-management-public-preview"></a>Nuevo flujo de trabajo de aprobación en dos fases de administración de derechos de Azure AD (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Administración de derechos

Hemos presentado un nuevo flujo de trabajo de aprobación en dos fases que le permite exigir dos aprobadores para aprobar la solicitud de un usuario a un paquete de acceso. Por ejemplo, puede establecerlo de manera que el administrador del usuario que realiza la solicitud deba dar su aprobación en primer lugar y, a continuación, también puede exigir que un propietario del recurso dé su aprobación. Si uno de los aprobadores no da su aprobación, no se concede el acceso.

Para obtener más información, consulte [Cambio de la configuración de la solicitud y aprobación para un paquete de acceso de administración de derechos de Azure AD](../governance/entitlement-management-access-package-request-policy.md).

---

### <a name="updates-to-the-my-apps-page-along-with-new-workspaces-public-preview"></a>Actualizaciones de la página Mis aplicaciones con nuevas áreas de trabajo (versión preliminar pública)

**Tipo:** Nueva característica  
**Categoría del servicio:** Mis aplicaciones  
**Funcionalidad del producto:** Integración de terceros

Ahora puede personalizar la forma en que los usuarios de la organización ven y acceden a la experiencia actualizada de Mis aplicaciones. Esta nueva experiencia también incluye la nueva característica de áreas de trabajo, que facilita a los usuarios la búsqueda y organización de aplicaciones.

Para obtener más información sobre la nueva experiencia de Mis aplicaciones y la creación de áreas de trabajo, consulte el artículo sobre [creación de áreas de trabajo en el portal Mis aplicaciones](../manage-apps/access-panel-collections.md).

---

### <a name="google-social-id-support-for-azure-ad-b2b-collaboration-general-availability"></a>Compatibilidad con identificadores sociales de Google para la colaboración B2B de Azure AD (disponibilidad general)

**Tipo:** Nueva característica  
**Categoría del servicio:** B2B  
**Funcionalidad del producto:** Autenticación de usuarios

La nueva compatibilidad con el uso de identificadores sociales de Google (cuentas Gmail) en Azure AD ayuda a simplificar la colaboración entre sus usuarios y partners. Ya no hay necesidad de que los partners creen y administren una cuenta específica de Microsoft. Microsoft Teams ahora es totalmente compatible con los usuarios de Google en todos los clientes y en los puntos de conexión de autenticación comunes y relacionados con los inquilinos.

Para obtener más información, consulte [Incorporación de Google como proveedor de identidades para los usuarios invitados de B2B](../external-identities/google-federation.md).

---

### <a name="microsoft-edge-mobile-support-for-conditional-access-and-single-sign-on-general-availability"></a>Compatibilidad de Microsoft Edge para dispositivos móviles con el acceso condicional y el inicio de sesión único (disponibilidad general)

**Tipo:** Nueva característica  
**Categoría del servicio:** Acceso condicional  
**Funcionalidad del producto:** Seguridad y protección de la identidad

Azure AD para Microsoft Edge en iOS y Android ahora admite acceso condicional e inicio de sesión único de Azure AD:

- **Inicio de sesión único (SSO) de Microsoft Edge:** El inicio de sesión único ahora está disponible en los clientes nativos (como Microsoft Outlook y Microsoft Edge) para todas las aplicaciones conectadas a Azure AD.

- **Acceso condicional de Microsoft Edge:** A través de las directivas de acceso condicional basado en aplicaciones, los usuarios deben usar exploradores protegidos por Microsoft Intune, como Microsoft Edge.

Para obtener más información sobre el acceso condicional y el inicio de sesión único con Microsoft Edge, consulte la entrada de blog sobre [compatibilidad de Microsoft Edge para dispositivos móviles con el acceso condicional y el inicio de sesión único ahora disponible con carácter general](https://techcommunity.microsoft.com/t5/Intune-Customer-Success/Microsoft-Edge-Mobile-Support-for-Conditional-Access-and-Single/ba-p/988179). Para obtener más información sobre cómo configurar las aplicaciones cliente con [acceso condicional basado en aplicaciones](../conditional-access/app-based-conditional-access.md) o [acceso condicional basado en dispositivos](../conditional-access/require-managed-devices.md), consulte [Administración del acceso web mediante un explorador protegido por directivas de Microsoft Intune](/intune/apps/app-configuration-managed-browser).

---

### <a name="azure-ad-entitlement-management-general-availability"></a>Administración de derechos de Azure AD (disponibilidad general)

**Tipo:** Nueva característica  
**Categoría del servicio:** Otros  
**Funcionalidad del producto:** Administración de derechos

La administración de derechos de Azure AD es una nueva característica de gobernanza de identidades, que ayuda a las organizaciones a administrar el ciclo de vida de identidad y acceso a escala. Esta nueva característica ayuda mediante la automatización de los flujos de trabajo de solicitud de acceso, las asignaciones de acceso, las revisiones y la expiración entre grupos, aplicaciones y sitios de SharePoint Online.

Con la administración de derechos de Azure AD, puede administrar el acceso de forma más eficaz tanto para los empleados como para los usuarios ajenos a la organización que han de acceder a esos recursos.

Para obtener más información, consulte [¿Qué es la administración de derechos de Azure AD?](../governance/entitlement-management-overview.md#license-requirements)

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatización del aprovisionamiento de cuentas de usuario para estas aplicaciones SaaS recién admitidas

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros  

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

[SAP Cloud Platform Identity Authentication Service](../saas-apps/sap-hana-cloud-platform-identity-authentication-tutorial.md), [RingCentral](../saas-apps/ringcentral-provisioning-tutorial.md), [SpaceIQ](../saas-apps/spaceiq-provisioning-tutorial.md), [Miro](../saas-apps/miro-provisioning-tutorial.md), [Cloudgate](../saas-apps/soloinsight-cloudgate-sso-provisioning-tutorial.md), [Infor CloudSuite](../saas-apps/infor-cloudsuite-provisioning-tutorial.md), [OfficeSpace Software](../saas-apps/officespace-software-provisioning-tutorial.md), [Priority Matrix](../saas-apps/priority-matrix-provisioning-tutorial.md)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---november-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: noviembre de 2019

**Tipo:** Nueva característica  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** Integración de terceros

En noviembre de 2019, hemos agregado estas 21 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[Airtable](../saas-apps/airtable-tutorial.md), [Hootsuite](../saas-apps/hootsuite-tutorial.md), [Blue Access for Members (BAM)](../saas-apps/blue-access-for-members-tutorial.md), [Bitly](../saas-apps/bitly-tutorial.md), [Riva](../saas-apps/riva-tutorial.md), [ResLife Portal](https://app.reslifecloud.com/hub5_signin/microsoft_azuread/?g=44BBB1F90915236A97502FF4BE2952CB&c=5&uid=0&ht=2&ref=), [NegometrixPortal Single Sign On (SSO)](../saas-apps/negometrixportal-tutorial.md), [TeamsChamp](https://login.microsoftonline.com/551f45da-b68e-4498-a7f5-a6e1efaeb41c/adminconsent?client_id=ca9bbfa4-1316-4c0f-a9ee-1248ac27f8ab&redirect_uri=https://admin.teamschamp.com/api/adminconsent&state=6883c143-cb59-42ee-a53a-bdb5faabf279), [Motus](../saas-apps/motus-tutorial.md), [MyAryaka](../saas-apps/myaryaka-tutorial.md), [BlueMail](https://loginself1.bluemail.me/), [Beedle](https://teams-web.beedle.co/#/), [Visma](../saas-apps/visma-tutorial.md), [OneDesk](../saas-apps/onedesk-tutorial.md), [Foko Retail](../saas-apps/foko-retail-tutorial.md), [Qmarkets Idea & Innovation Management](../saas-apps/qmarkets-idea-innovation-management-tutorial.md), [Netskope User Authentication](../saas-apps/netskope-user-authentication-tutorial.md), [uniFLOW Online](../saas-apps/uniflow-online-tutorial.md), [Claromentis](../saas-apps/claromentis-tutorial.md), [Jisc Student Voter Registration](../saas-apps/jisc-student-voter-registration-tutorial.md), [e4enable](https://portal.e4enable.com/)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-and-improved-azure-ad-application-gallery"></a>La nueva y mejorada galería de aplicaciones de Azure AD

**Tipo:** Característica modificada  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO

Hemos actualizado la galería de aplicaciones de Azure AD para que le resulte más fácil encontrar aplicaciones integradas previamente que admitan el aprovisionamiento, OpenID Connect y SAML en su inquilino de Azure Active Directory.

Para obtener más información, consulte [Incorporación de una aplicación a un inquilino de Azure Active Directory](../manage-apps/add-application-portal.md).

---

### <a name="increased-app-role-definition-length-limit-from-120-to-240-characters"></a>Límite de longitud de definición de roles de aplicación aumentado de 120 a 240 caracteres

**Tipo:** Característica modificada  
**Categoría del servicio:** Aplicaciones empresariales  
**Funcionalidad del producto:** SSO

Hemos escuchado de los clientes que el límite de longitud del valor de definición de rol de aplicación en algunas aplicaciones y servicios es demasiado corto en 120 caracteres. En respuesta, hemos aumentado la longitud máxima de la definición de valor de rol a 240 caracteres.

Para obtener más información sobre el uso de definiciones de roles específicas de la aplicación, consulte Para ver más ejemplos e información, consulte [Procedimiento para agregar roles de aplicación en la aplicación y recibirlos en el token](../develop/howto-add-app-roles-in-azure-ad-apps.md).

---

## <a name="october-2019"></a>Octubre de 2019

### <a name="deprecation-of-the-identityriskevent-api-for-azure-ad-identity-protection-risk-detections"></a>Desuso de la API de identityRiskEvent para las detecciones de riesgo de Azure AD Identity Protection

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

En respuesta a los comentarios de los desarrolladores, los suscriptores de Azure AD Premium P2 ahora pueden realizar consultas complejas sobre los datos de detección de riesgos de Azure AD Identity Protection mediante la nueva API de riskDetection para Microsoft Graph. La versión beta existente de la API de [identityRiskEvent](/graph/api/resources/identityriskevent?view=graph-rest-beta&preserve-view=true) dejará de devolver datos en torno al **10 de enero de 2020**. Si su organización usa la API de identityRiskEvent, debe realizar la transición a la nueva API de riskDetection.

Para obtener más información sobre la nueva API de riskDetection, consulte la [documentación de referencia de la API de detección de riesgos](/graph/api/resources/riskdetection).

---

### <a name="application-proxy-support-for-the-samesite-attribute-and-chrome-80"></a>Compatibilidad del proxy de aplicación con el atributo SameSite y Chrome 80

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Un par de semanas antes del lanzamiento del explorador Chrome 80, tenemos previsto actualizar el modo en que las cookies del proxy de aplicación tratan el atributo **SameSite**. Con el lanzamiento de Chrome 80, cualquier cookie que no especifique el atributo **SameSite** se tratará como si se hubiera establecido en `SameSite=Lax`.

Para ayudar a evitar posibles impactos negativos debido a este cambio, vamos a actualizar las cookies de sesión y el acceso del proxy de aplicación de la siguiente manera:

- Estableciendo el valor predeterminado de la opción **Usar cookies seguras** en **Sí**.

- Estableciendo el valor predeterminado del atributo **SameSite** en **Ninguno**.

    >[!NOTE]
    > Las cookies de acceso del proxy de aplicación siempre se han transmitido exclusivamente a través de canales seguros. Estos cambios solo se aplican a las cookies de sesión.

Para obtener más información sobre la configuración de las cookies del proxy de aplicación, consulte [Configuración de las cookies para el acceso a aplicaciones locales en Azure Active Directory](../manage-apps/application-proxy-configure-cookie-settings.md).

---

### <a name="app-registrations-legacy-and-app-management-in-the-application-registration-portal-appsdevmicrosoftcom-is-no-longer-available"></a>La característica Registros de aplicaciones (heredada) y la funcionalidad de administración de aplicaciones en el Portal de registro de aplicaciones (apps.dev.microsoft.com) ya no están disponibles

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** N/D Experiencia para el desarrollador

Los usuarios con cuentas de Azure AD ya no pueden registrar o administrar aplicaciones mediante el Portal de registro de aplicaciones (apps.dev.microsoft.com), ni registrar ni administrar aplicaciones en la experiencia Registros de aplicaciones (característica heredada) de Azure Portal.

Para obtener más información sobre la nueva experiencia Registros de aplicaciones, consulte la sección sobre [Registros de aplicaciones en la guía de aprendizaje de Azure Portal](../develop/quickstart-register-app.md).

---

### <a name="users-are-no-longer-required-to-re-register-during-migration-from-per-user-mfa-to-conditional-access-based-mfa"></a>Ya no es necesario que los usuarios vuelvan a registrarse durante la migración de MFA por usuario a MFA basada en el acceso condicional

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** MFA Seguridad y protección de la identidad

Hemos corregido un problema conocido por el que se requería que los usuarios se volvieran a registrar si estaban deshabilitados para la autenticación multifactor (MFA) por usuario y luego se habilitaban para la MFA a través de una directiva de acceso condicional.

Para requerir que los usuarios vuelvan a registrarse, puede seleccionar la opción **Requerir volver a registrar MFA** en los métodos de autenticación del usuario en el portal de Azure AD. Para obtener más información sobre la migración de usuarios de MFA por usuario a MFA basada en el acceso condicional, consulte [Conversión de los usuarios de MFA por usuario a MFA basado en acceso condicional](../authentication/howto-mfa-getstarted.md#convert-users-from-per-user-mfa-to-conditional-access-based-mfa).

---

### <a name="new-capabilities-to-transform-and-send-claims-in-your-saml-token"></a>Nuevas funcionalidades para transformar y enviar notificaciones en el token SAML

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Hemos agregado funcionalidades adicionales que lo ayudarán a personalizar y enviar notificaciones en el token SAML. Estas son algunas de ellas:

- Funciones de transformación de notificaciones adicionales, lo que lo ayudará a modificar el valor que envía en la notificación.

- Capacidad de aplicar varias transformaciones a una única notificación.

- Capacidad de especificar el origen de la notificación, en función del tipo de usuario y el grupo al que pertenece el usuario.

Para obtener información detallada sobre estas nuevas funcionalidades, incluido cómo usarlas, consulte [Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales](../develop/active-directory-saml-claims-customization.md).

---

### <a name="new-my-sign-ins-page-for-end-users-in-azure-ad"></a>Nueva página Mis inicios de sesión para los usuarios finales en Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Supervisión e informes

Hemos agregado una nueva página **Mis inicios de sesión** (https://mysignins.microsoft.com) ) para que los usuarios de la organización puedan ver su historial de inicio de sesión reciente con el fin de comprobar si hay alguna actividad inusual. Esta nueva página permite a los usuarios comprobar lo siguiente:

- Si alguien intenta adivinar la contraseña.

- Si un atacante inició sesión correctamente en su cuenta y desde qué ubicación.

- Las aplicaciones a las que el atacante ha intentado acceder.

Para obtener más información, consulte la entrada de blog sobre cómo [los usuarios pueden comprobar ahora su historial de inicio de sesión para actividades inusuales](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Users-can-now-check-their-sign-in-history-for-unusual-activity/ba-p/916066).

---

### <a name="migration-of-azure-ad-domain-services-azure-ad-ds-from-classic-to-azure-resource-manager-virtual-networks"></a>Migración de Azure AD Domain Services (Azure AD DS) de las redes virtuales clásicas a las de Azure Resource Manager

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Tenemos excelentes noticias para los clientes que siguen usando las redes virtuales clásicas. Ahora puede realizar una migración puntual desde una red virtual clásica a una de Resource Manager. Después de migrar a la red virtual de Resource Manager, podrá aprovechar las ventajas de las características adicionales y actualizadas, como las directivas de contraseñas específicas, las notificaciones por correo electrónico y los registros de auditoría.

Para obtener más información, consulte [Versión preliminar: Migración de Azure AD Domain Services desde el modelo de red virtual clásica a Resource Manager](../../active-directory-domain-services/migrate-from-classic-vnet.md).

---

### <a name="updates-to-the-azure-ad-b2c-page-contract-layout"></a>Actualizaciones del diseño del contrato de página de Azure AD B2C

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Hemos introducido algunos cambios nuevos en la versión 1.2.0 del contrato de página de Azure AD B2C. En esta versión actualizada, ahora puede controlar el orden de carga de los elementos, lo que también puede ayudar a detener el parpadeo que se produce cuando se carga la hoja de estilos (CSS).

Para ver una lista completa de los cambios realizados en el contrato de página, vea el [registro de cambios de versión](../../active-directory-b2c/page-layout.md#other-pages-providerselection-claimsconsent-unifiedssd).

---

### <a name="update-to-the-my-apps-page-along-with-new-workspaces-public-preview"></a>Actualización de la página Mis aplicaciones con nuevas áreas de trabajo (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones Control de acceso

Ahora puede personalizar la forma en que los usuarios de su organización ven y acceden a la nueva experiencia de Mis aplicaciones, incluido el uso de la nueva característica de áreas de trabajo para que sea más fácil buscar aplicaciones. La nueva funcionalidad de áreas de trabajo actúa como un filtro para las aplicaciones a las que los usuarios de su organización ya tienen acceso.

Para obtener más información sobre cómo implementar la nueva experiencia de Mis aplicaciones y crear áreas de trabajo, consulte el artículo sobre [creación de áreas de trabajo en el portal Mis aplicaciones (versión preliminar)](../manage-apps/access-panel-collections.md).

---

### <a name="support-for-the-monthly-active-user-based-billing-model-general-availability"></a>Compatibilidad con el modelo de facturación basado en usuarios activos mensuales (disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Azure AD B2C admite ahora la facturación de usuarios activos mensuales (MAU). La facturación de MAU se basa en el número de usuarios únicos con actividad de autenticación durante un mes natural. Los clientes existentes pueden pasarse a este nuevo método de facturación en cualquier momento.

A partir del 1 de noviembre de 2019, a todos los clientes nuevos se les facturará automáticamente con este método. Este método de facturación beneficia a los clientes gracias a las ventajas que ofrece en cuanto a costos y a la posibilidad de planear con anterioridad.

Para obtener más información, consulte [Actualización al modelo de facturación de usuarios activos mensuales](../../active-directory-b2c/billing.md#switch-to-mau-billing-pre-november-2019-azure-ad-b2c-tenants).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---october-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: octubre de 2019

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En octubre de 2019, hemos agregado estas 35 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[In Case of Crisis – Mobile](../saas-apps/in-case-of-crisis-mobile-tutorial.md), [Juno Journey](../saas-apps/juno-journey-tutorial.md), [ExponentHR](../saas-apps/exponenthr-tutorial.md), [Tact](https://www.tact.ai/products/tact-assistant), [OpusCapita Cash Management](https://appsource.microsoft.com/product/web-apps/opuscapitagroupoy-1036255.opuscapita-cm), [Salestim](https://www.salestim.com/), [Learnster](../saas-apps/learnster-tutorial.md), [Dynatrace](../saas-apps/dynatrace-tutorial.md), [HunchBuzz](https://login.hunchbuzz.com/integrations/azure/process), [Freshworks](../saas-apps/freshworks-tutorial.md), [eCornell](../saas-apps/ecornell-tutorial.md), [ShipHazmat](../saas-apps/shiphazmat-tutorial.md), [Netskope Cloud Security](../saas-apps/netskope-cloud-security-tutorial.md), [Contentful](../saas-apps/contentful-tutorial.md), [Bindtuning](https://bindtuning.com/login), [HireVue Coordinate – Europe](https://www.hirevue.com/), [HireVue Coordinate - USOnly](https://www.hirevue.com/), [HireVue Coordinate - US](https://www.hirevue.com/), [WittyParrot Knowledge Box](https://wittyapi.wittyparrot.com/wittyparrot/api/provision/trail/signup), [Cloudmore](../saas-apps/cloudmore-tutorial.md), [Visit.org](../saas-apps/visitorg-tutorial.md), [Cambium Xirrus EasyPass Portal](https://login.xirrus.com/azure-signup), [Paylocity](../saas-apps/paylocity-tutorial.md), [Mail Luck!](../saas-apps/mail-luck-tutorial.md), [Teamie](https://theteamie.com/), [Velocity for Teams](https://velocity.peakup.org/teams/login), [SIGNL4](https://account.signl4.com/manage), [EAB Navigate IMPL](../saas-apps/eab-navigate-impl-tutorial.md), [ScreenMeet](https://console.screenmeet.com/), [Omega Point](https://pi.ompnt.com/), [Speaking Email for Intune (iPhone)](https://speaking.email/FAQ/98/email-access-via-microsoft-intune), [Speaking Email for Office 365 Direct (iPhone/Android)](https://speaking.email/FAQ/126/email-access-via-microsoft-office-365-direct), [ExactCare SSO](../saas-apps/exactcare-sso-tutorial.md), [iHealthHome Care Navigation System](https://ihealthnav.com/account/signin), [Qubie](https://qubie.azurewebsites.net/static/adminTab/authorize.html)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="consolidated-security-menu-item-in-the-azure-ad-portal"></a>Elemento de menú consolidado Seguridad en el portal de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

Ahora puede acceder a todas las características de seguridad de Azure AD disponibles en el nuevo elemento de menú **Seguridad** y en la barra **Buscar** de Azure Portal. Además, la nueva página de aterrizaje **Seguridad** denominada **Seguridad - Introducción** proporcionará vínculos a nuestra documentación pública, así como guías de seguridad y de implementación.

El nuevo menú **Seguridad** incluye lo siguiente:

- Acceso condicional
- Protección de identidad
- Security Center
- Puntuación segura de identidad
- Métodos de autenticación
- MFA
- Informes de riesgo: usuarios de riesgo, inicios de sesión peligrosos, detecciones de riesgo, etc.
- Y mucho más.

Para obtener más información, visite [Seguridad - Introducción](https://portal.azure.com/#blade/Microsoft_AAD_IAM/SecurityMenuBlade/GettingStarted).

---

### <a name="office-365-groups-expiration-policy-enhanced-with-autorenewal"></a>Directiva de expiración de grupos de Office 365 mejorada con renovación automática

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Administración de grupos Administración del ciclo de vida de la identidad

La directiva de expiración de grupos de Office 365 se ha mejorado para renovar automáticamente los grupos que los miembros estén usando activamente. Los grupos se renuevan en función de la actividad del usuario en todas las aplicaciones de Office 365, como Outlook, SharePoint y Teams.

Esta mejora ayuda a reducir las notificaciones de expiración de los grupos, así como a garantizar que los grupos activos sigan estando disponibles. Si ya tiene una directiva de expiración activa para grupos de Office 365, no es necesario hacer nada para activar esta nueva funcionalidad.

Para obtener más información, vea [Configuración de la directiva de expiración de grupos de Office 365](../enterprise-users/groups-lifecycle.md).

---

### <a name="updated-azure-ad-domain-services-azure-ad-ds-creation-experience"></a>Experiencia de creación de Azure AD Domain Services (Azure AD DS) actualizada

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Hemos actualizado Azure AD Domain Services (Azure AD DS) para incluir una experiencia de creación nueva y mejorada, que lo ayudará a crear un dominio administrado en tan solo tres clics. Además, ahora puede cargar e implementar instancias de Azure AD DS desde una plantilla.

Para más información, consulte el [Tutorial: Creación y configuración de una instancia de Azure Active Directory Domain Services](../../active-directory-domain-services/tutorial-create-instance.md).

---

## <a name="september-2019"></a>Septiembre de 2019

### <a name="plan-for-change-deprecation-of-the-power-bi-content-packs"></a>Planeado su cambio: Desuso de los paquetes de contenido de Power BI

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto**: Informes Supervisión e informes

A partir del 1 de octubre de 2019, Power BI comenzará a dejar de usar todos los paquetes de contenido, incluido el paquete de contenido de Azure AD Power BI. Como alternativa a este paquete de contenido, puede usar libros de Azure AD para obtener información sobre los servicios relacionados con Azure AD. Próximos libros de trabajo adicionales, incluidos los libros sobre las directivas de acceso condicional en modo de solo informes, información basada en el consentimiento de la aplicación, etc.

Para obtener más información acerca de los libros, consulte [Cómo usar los libros de Azure Monitor en informes de Azure Active Directory](../reports-monitoring/howto-use-azure-monitor-workbooks.md). Para obtener más información sobre el desuso de los paquetes de contenido, consulte la publicación de blog [Anuncio de disponibilidad general de aplicaciones de plantilla de Power BI](https://powerbi.microsoft.com/blog/announcing-power-bi-template-apps-general-availability/).

---

### <a name="my-profile-is-renaming-and-integrating-with-the-microsoft-office-account-page"></a>Mi perfil cambia de nombre y se integra con la página de la cuenta de Microsoft Office

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto**: Mi perfil/cuenta Colaboración

A partir de octubre, la experiencia Mi perfil se convertirá en Mi cuenta. Como parte de ese cambio, en todas partes en las que aparezca **Mi perfil**, este pasará a llamarse **Mi cuenta**. Además del cambio de nomenclatura y algunas mejoras de diseño, la experiencia actualizada ofrecerá una integración adicional con la página de la cuenta de Microsoft Office. En concreto, podrá obtener acceso a las instalaciones y suscripciones de Office desde la página **Overview Account** (Descripción general de la cuenta), junto con las preferencias de contacto relacionadas con Office desde la página **Privacy** (Privacidad).

Para obtener más información sobre la experiencia Mi perfil (versión preliminar), consulte la [descripción general del portal Mi perfil (versión preliminar)](../user-help/my-account-portal-overview.md).

---

### <a name="bulk-manage-groups-and-members-using-csv-files-in-the-azure-ad-portal-public-preview"></a>Administración masiva de grupos y miembros mediante archivos CSV en el portal de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Nos complace anunciar la disponibilidad de la versión preliminar pública de las experiencias de administración masiva de grupos en el portal de Azure AD. Ahora puede usar un archivo CSV y el portal de Azure AD para administrar grupos y listas de miembros, entre los que se incluyen:

- Incorporación o eliminación de miembros de un grupo.

- Descarga de la lista de grupos del directorio.

- Descarga de la lista de miembros del grupo para un grupo específico.

Para obtener más información, consulte [Agregar miembros de forma masiva](../enterprise-users/groups-bulk-import-members.md), [Eliminar miembros de forma masiva](../enterprise-users/groups-bulk-remove-members.md), [Descarga masiva de los miembros de la lista](../enterprise-users/groups-bulk-download-members.md) y [Descarga masiva de los grupos de la lista](../enterprise-users/groups-bulk-download.md).

---

### <a name="dynamic-consent-is-now-supported-through-a-new-admin-consent-endpoint"></a>Además se admite el consentimiento dinámico a través de un nuevo punto de conexión de consentimiento del administrador.

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Hemos creado un nuevo punto de conexión de consentimiento del administrador para admitir la opción de consentimiento dinámico, lo que le resultará útil para las aplicaciones que vayan a usar el modelo de consentimiento dinámico en la plataforma de Microsoft Identity.

Para obtener más información sobre cómo usar este nuevo punto de conexión, consulte [Usar el punto de conexión del consentimiento de administrador](../develop/v2-admin-consent.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---september-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: septiembre de 2019

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En septiembre de 2019, hemos agregado estas 29 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[ScheduleLook](https://schedulelook.bbsonlineservices.net/), [MS Azure SSO Access for Ethidex Compliance Office&trade; - Single  sign-on](../saas-apps/ms-azure-sso-access-for-ethidex-compliance-office-tutorial.md), [iServer Portal](../saas-apps/iserver-portal-tutorial.md), [SKYSITE](../saas-apps/skysite-tutorial.md), [Concur Travel and Expense](../saas-apps/concur-travel-and-expense-tutorial.md), [WorkBoard](../saas-apps/workboard-tutorial.md), `https://apps.yeeflow.com/`, [ARC Facilities](../saas-apps/arc-facilities-tutorial.md), [Luware Stratus Team](https://stratus.emea.luware.cloud/login), [Wide Ideas](https://wideideas.online/wideideas/), [Prisma Cloud](../saas-apps/prisma-cloud-tutorial.md), [JDLT Client Hub](https://clients.jdlt.co.uk/login), [RENRAKU](../saas-apps/renraku-tutorial.md), [SealPath Secure Browser](https://protection.sealpath.com/SealPathInterceptorWopiSaas/Open/InstallSealPathEditorOneDrive), [Prisma Cloud](../saas-apps/prisma-cloud-tutorial.md), `https://app.penneo.com/`, `https://app.testhtm.com/settings/email-integration`, [Cintoo Cloud](https://aec.cintoo.com/login), [Whitesource](../saas-apps/whitesource-tutorial.md), [Hosted Heritage Online SSO](../saas-apps/hosted-heritage-online-sso-tutorial.md), [IDC](../saas-apps/idc-tutorial.md), [CakeHR](../saas-apps/cakehr-tutorial.md), [BIS](../saas-apps/bis-tutorial.md), [Coo Kai Team Build](https://ms-contacts.coo-kai.jp/), [Sonarqube](../saas-apps/sonarqube-tutorial.md), [Adobe Identity Management](../saas-apps/tutorial-list.md), [Discovery Benefits SSO](../saas-apps/discovery-benefits-sso-tutorial.md), [Amelio](https://app.amelio.co/), `https://itask.yipinapp.com/`

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-azure-ad-global-reader-role"></a>Nuevo rol de lector global de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto de roles de Azure AD:** Control de acceso

A partir del 24 de septiembre de 2019, empezaremos a implementar un nuevo rol de Azure Active Directory (AD) denominado Lector global. Este lanzamiento comenzará con los clientes de producción y de la nube global (GCC), y finalizará en octubre a nivel global.

El rol del Lector global es la contrapartida de la opción de solo lectura del administrador global. Los usuarios de este rol pueden leer la configuración y la información administrativa en los servicios de Microsoft 365, pero no pueden llevar a cabo acciones de administración. Por ello, hemos creado el rol del Lector global para que pueda reducir la cantidad de administradores globales en su organización. Debido a que las cuentas de administrador global son a la vez eficaces y vulnerables a los ataques, le recomendamos que tenga menos de cinco administradores globales. Igualmente, le recomendamos usar el rol de Lector global para la planificación, las auditorías o las investigaciones. También le recomendamos usar el rol de Lector global en combinación con otros roles de administrador limitados, como el de administrador de Exchange, para que pueda realizar el trabajo necesario sin tener que usar el rol de administrador global.

El rol del Lector global funciona con el nuevo Centro de administración de Microsoft 365, el Centro de administración de Exchange, el Centro de administración de Teams, el Centro de seguridad, el Centro de cumplimiento, el Centro de administración de Azure AD y el Centro de administración de la administración de dispositivos.

>[!NOTE]
> Al comienzo de la versión preliminar pública, el rol de Lector global no funcionará con: SharePoint, Privileged Access Management, Caja de seguridad del cliente, etiquetas de confidencialidad, Ciclo de vida de Teams, Creación de informes y análisis de llamadas de Teams, Administración de dispositivos de teléfono IP de Teams y Catálogo de aplicaciones de Teams.

Para obtener más información, consulte los [permisos del rol de administrador en Azure Active Directory](../roles/permissions-reference.md).

---

### <a name="access-an-on-premises-report-server-from-your-power-bi-mobile-app-using-azure-active-directory-application-proxy"></a>Acceder a un servidor de informes local desde la aplicación Power BI Mobile con Azure Active Directory Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

La nueva integración entre la aplicación Power BI Mobile y Azure AD Application Proxy le permite iniciar sesión de forma segura en la aplicación Power BI Mobile y ver cualquiera de los informes de la organización hospedados en la instancia local de Power BI Report Server.

Para obtener más información acerca de la aplicación de Power BI Mobile, incluyendo el lugar desde dónde descargar la aplicación, consulte el [sitio de Power BI](https://powerbi.microsoft.com/mobile/). Para obtener más información sobre cómo configurar la aplicación de Power BI Mobile con Azure AD Application Proxy, consulte [Habilitar el acceso remoto a Power BI Mobile con Azure AD Application Proxy](../manage-apps/application-proxy-integrate-with-power-bi.md).

---

### <a name="new-version-of-the-azureadpreview-powershell-module-is-available"></a>Hay una nueva versión del módulo de PowerShell de AzureADPreview disponible

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Directorio

Se agregaron nuevos cmdlets al módulo AzureADPreview, para ayudar a definir y asignar roles personalizados en Azure AD, que incluyen:

- `Add-AzureADMSFeatureRolloutPolicyDirectoryObject`
- `Get-AzureADMSFeatureRolloutPolicy`
- `New-AzureADMSFeatureRolloutPolicy`
- `Remove-AzureADMSFeatureRolloutPolicy`
- `Remove-AzureADMSFeatureRolloutPolicyDirectoryObject`
- `Set-AzureADMSFeatureRolloutPolicy`

---

### <a name="new-version-of-azure-ad-connect"></a>Nueva versión de Azure AD Connect

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Directorio

Hemos publicado una versión actualizada de Azure AD Connect para clientes que usen la actualización automática. Esta nueva versión incluye varias características nuevas, mejoras y correcciones de errores. Para obtener más información sobre esta versión nueva, consulte [Azure AD Connect: historial de versiones](../hybrid/reference-connect-version-history.md#14250).

---

### <a name="azure-multi-factor-authentication-mfa-server-version-802-is-now-available"></a>Ya está disponible la versión 8.0.2 del servidor de Multi-Factor Authentication de Azure (MFA)

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** MFA Seguridad y protección de la identidad

Si es un cliente ya existente que activó el servidor MFA antes del 1 de julio de 2019, ya puede descargar la versión más reciente del servidor MFA (versión 8.0.2). En esta nueva versión, hemos realizado lo siguiente:

- Se corrigió un problema por el que, cuando Azure AD Sync cambia el estado de un usuario de "deshabilitado" a "habilitado", se envía un correo electrónico al mismo.

- Se corrigió un problema para que los clientes puedan realizar sus actualizaciones correctamente, al tiempo que se sigue usando la funcionalidad de etiquetas.

- Se agregó el código de país de Kosovo (+ 383).

- Se agregó un registro de auditoría de omisión por única vez a MultiFactorAuthSvc.log.

- Se mejoró el rendimiento del SDK del servicio web.

- Se corrigieron otros errores menores.

A partir del 1 de julio de 2019, Microsoft dejó de ofrecer el servidor de MFA en las nuevas implementaciones. Los clientes nuevos que quieran exigir la autenticación multifactor a sus usuarios deberán usar Azure AD Multi-Factor Authentication basado en la nube. Para obtener más información, consulte [Planeamiento de una implementación de Azure AD Multi-Factor Authentication basada en la nube](../authentication/howto-mfa-getstarted.md).

---

## <a name="august-2019"></a>Agosto de 2019

### <a name="enhanced-search-filtering-and-sorting-for-groups-is-available-in-the-azure-ad-portal-public-preview"></a>Funciones de búsqueda, filtrado y ordenación mejorados para los grupos están disponibles en el portal de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Nos complace anunciar la disponibilidad de la versión preliminar pública de las experiencias mejoradas relacionadas con grupos en el portal de Azure AD. Estas mejoras ayudan a administrar mejor los grupos y las listas de miembros, ya que proporcionan:

- Funcionalidades de búsqueda avanzada, como la búsqueda de subcadenas en listas de grupos.
- Opciones avanzadas de filtrado y ordenación en las listas de miembros y propietarios.
- Nuevas funcionalidades de búsqueda para listas de miembros y propietarios.
- Recuentos de grupos más precisos para grupos grandes.

Para obtener más información, consulte [Administración de grupos en Azure Portal](./active-directory-groups-members-azure-portal.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context).

---

### <a name="new-custom-roles-are-available-for-app-registration-management-public-preview"></a>Nuevos roles personalizados están disponibles para la administración de registro de aplicaciones (Versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto de roles de Azure AD:** Control de acceso

Los roles personalizados (disponibles con una suscripción Azure AD P1 o P2) ahora pueden ayudarle a proporcionar acceso específico, permitiéndole crear definiciones de roles con permisos específicos y, a continuación, asignar esos roles a recursos específicos. Actualmente, se crean roles personalizados mediante el uso de permisos para administrar los registros de aplicaciones y, a continuación, se asigna el rol a una aplicación específica. Para obtener más información sobre los roles personalizados, consulte [Roles de administrador personalizados en Azure Active Directory (versión preliminar)](../roles/custom-overview.md).

Si necesita más permisos o recursos admitidos que no vea actualmente, puede enviar comentarios a nuestro [sitio de comentarios de Azure](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032) e incorporaremos su solicitud a nuestra guía de ruta de actualización.

---

### <a name="new-provisioning-logs-can-help-you-monitor-and-troubleshoot-your-app-provisioning-deployment-public-preview"></a>Los nuevos registros de aprovisionamiento pueden ayudarle a supervisar y solucionar problemas de la implementación de aprovisionamiento de aplicaciones (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Administración del ciclo de vida de la identidad

Nuevos registros de aprovisionamiento disponibles para ayudarle a supervisar y solucionar problemas de la implementación de aprovisionamiento de usuarios y grupos. Estos nuevos archivos de registro incluyen información sobre:

- Qué grupos se han creado correctamente en [ServiceNow](../saas-apps/servicenow-provisioning-tutorial.md)
- ¿Qué roles se importaron desde [AWS Single-Account Access](../saas-apps/amazon-web-service-tutorial.md#configure-and-test-azure-ad-sso-for-aws-single-account-access)
- Qué empleados no se importaron desde [WorkDay](../saas-apps/workday-inbound-tutorial.md)

Para obtener más información, consulte [Informes de aprovisionamiento en el portal de Azure Active Directory (versión preliminar)](../reports-monitoring/concept-provisioning-logs.md).

---

### <a name="new-security-reports-for-all-azure-ad-administrators-general-availability"></a>Nuevos informes de seguridad para todos los administradores de Azure AD (Disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

De forma predeterminada, todos los administradores de Azure AD podrán acceder pronto a informes de seguridad modernos dentro de Azure AD. Hasta el final de septiembre, podrá usar el banner de la parte superior de los informes de seguridad modernos para volver a los informes antiguos.

Los informes de seguridad modernos proporcionarán funciones adicionales de las versiones anteriores, entre las que se incluyen:

- Filtrado y ordenación avanzados
- Acciones en masa, como descartar el riesgo del usuario
- Confirmación de entidades en peligro o seguras
- Estado de riesgo, que abarca: En riesgo, Descartado, Corregido y Confirmado (en peligro)
- Nuevas detecciones relacionadas con riesgos (disponibles para suscriptores de Azure AD Premium)

Para obtener más información, consulte [ Usuarios riesgosos ](../identity-protection/howto-identity-protection-investigate-risk.md#risky-users), [ Inicios de sesión riesgosos ](../identity-protection/howto-identity-protection-investigate-risk.md#risky-sign-ins) y [ Detecciones de riesgos ](../identity-protection/howto-identity-protection-investigate-risk.md#risk-detections).

---

### <a name="user-assigned-managed-identity-is-available-for-virtual-machines-and-virtual-machine-scale-sets-general-availability"></a>La identidad administrada asignada por el usuario está disponible para máquinas virtuales y conjuntos de escalado de máquinas virtuales (Disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identidades administradas para recursos de Azure Experiencia para el desarrollador

Las identidades administradas asignadas por el usuario ahora están generalmente disponibles para máquinas virtuales y conjuntos de escalado de máquinas virtuales. Como parte de esto, Azure puede crear una identidad en el inquilino de Azure AD que sea de confianza para la suscripción en uso y se puede asignar a una o varias instancias de servicio de Azure. Para obtener más información sobre las identidades administradas asignadas por el usuario, consulte [ ¿Qué son las identidades administradas para los recursos de Azure? ](../managed-identities-azure-resources/overview.md).

---

### <a name="users-can-reset-their-passwords-using-a-mobile-app-or-hardware-token-general-availability"></a>Los usuarios pueden restablecer sus contraseñas mediante una aplicación móvil o un token de hardware (Disponibilidad general)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Autenticación de usuarios

Los usuarios que han registrado una aplicación móvil en su organización ahora pueden restablecer su propia contraseña mediante la aprobación de una notificación de la aplicación de Microsoft Authenticator o mediante la introducción de un código de su aplicación móvil o token de hardware.

Para obtener más información, [Consulte cómo funciona: Autoservicio de restablecimiento de contraseña de Azure AD](../authentication/concept-sspr-howitworks.md). Para obtener más información sobre la experiencia del usuario, consulte [Información general sobre cómo restablecer su propia contraseña profesional o educativa](../user-help/active-directory-passwords-reset-register.md).

---

### <a name="adalnet-ignores-the-msalnet-shared-cache-for-on-behalf-of-scenarios"></a>ADAL.NET ignora la memoria caché compartida de MSAL.NET para escenarios en representación

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

A partir de la versión 5.0.0-preview de la biblioteca de autenticación de Azure AD (ADAL.NET), los desarrolladores de aplicaciones deben [serializar una caché por cuenta para las aplicaciones web y API web](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Token-cache-serialization#custom-token-cache-serialization-in-web-applications--web-api). En caso contrario, algunos escenarios que usan el [flujo con derechos delegados](../develop/scenario-web-api-call-api-app-configuration.md?tabs=java) para Java, junto con algunos casos de uso específicos de `UserAssertion`, pueden dar como resultado una elevación de privilegios. Para evitar esta vulnerabilidad, ADAL.NET ignora ahora la caché compartida de la biblioteca de autenticación de Microsoft para dotnet (MSAL.NET) para escenarios en representación.

Para más información acerca de este problema, consulte [Vulnerabilidad de elevación de privilegios de la Biblioteca de autenticación de Active Directory](https://portal.msrc.microsoft.com/security-guidance/advisory/CVE-2019-1258).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---august-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: agosto de 2019

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En agosto de 2019, hemos agregado estas 26 nuevas aplicaciones con compatibilidad con la federación para la galería de aplicaciones:

[Civic Platform](../saas-apps/civic-platform-tutorial.md), [Amazon Business](../saas-apps/amazon-business-tutorial.md), [ProNovos Ops Manager](../saas-apps/pronovos-ops-manager-tutorial.md), [Cognidox](../saas-apps/cognidox-tutorial.md), [Viareport's Inativ Portal (Europe)](../saas-apps/viareports-inativ-portal-europe-tutorial.md), [Azure Databricks](https://azure.microsoft.com/services/databricks), [Robin](../saas-apps/robin-tutorial.md), [Academy Attendance](../saas-apps/academy-attendance-tutorial.md), [Priority Matrix](https://sync.appfluence.com/pmwebng/), [Cousto MySpace](https://cousto.platformers.be/account/login), [Uploadcare](https://uploadcare.com/accounts/signup/), [Carbonite Endpoint Backup](../saas-apps/carbonite-endpoint-backup-tutorial.md), [CPQSync by Cincom](../saas-apps/cpqsync-by-cincom-tutorial.md), [Chargebee](../saas-apps/chargebee-tutorial.md), [deliver.media&trade; Portal](https://portal.deliver.media), [Frontline Education](../saas-apps/frontline-education-tutorial.md), [F5](https://www.f5.com/products/security/access-policy-manager), [stashcat AD connect](https://www.stashcat.com), [Blink](../saas-apps/blink-tutorial.md), [Vocoli](../saas-apps/vocoli-tutorial.md), [ProNovos Analytics](../saas-apps/pronovos-analytics-tutorial.md), [Sigstr](../saas-apps/sigstr-tutorial.md), [Darwinbox](../saas-apps/darwinbox-tutorial.md), [Watch by Colors](../saas-apps/watch-by-colors-tutorial.md), [Harness](../saas-apps/harness-tutorial.md), [EAB Navigate Strategic Care](../saas-apps/eab-navigate-strategic-care-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-versions-of-the-azuread-powershell-and-azureadpreview-powershell-modules-are-available"></a>Están disponibles nuevas versiones de los módulos AzureAD PowerShell y AzureADPreview PowerShell.

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Directorio

Están disponibles nuevas versiones de los módulos AzureAD PowerShell y AzureADPreview PowerShell.

- Se agregó un nuevo parámetro `-Filter` al parámetro `Get-AzureADDirectoryRole` en el módulo AzureAD. Este parámetro le ayuda a filtrar por los roles de directorio devueltos por el cmdlet.
- Se agregaron nuevos cmdlets al módulo AzureADPreview, para ayudar a definir y asignar roles personalizados en Azure AD, que incluyen:

    - `Get-AzureADMSRoleAssignment`
    - `Get-AzureADMSRoleDefinition`
    - `New-AzureADMSRoleAssignment`
    - `New-AzureADMSRoleDefinition`
    - `Remove-AzureADMSRoleAssignment`
    - `Remove-AzureADMSRoleDefinition`
    - `Set-AzureADMSRoleDefinition`

---

### <a name="improvements-to-the-ui-of-the-dynamic-group-rule-builder-in-the-azure-portal"></a>Mejoras en la interfaz de usuario del generador de reglas de grupos dinámicos en Azure Portal

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Administración de grupos Colaboración

Hemos realizado algunas mejoras de la interfaz de usuario en el generador de reglas de grupo dinámico, disponible en el Azure Portal, para ayudarle a configurar más fácilmente una nueva regla o cambiar las reglas existentes. Esta mejora del diseño le permite crear reglas con hasta cinco expresiones, en lugar de solo una. También hemos actualizado la lista de propiedades del dispositivo para quitar las propiedades del dispositivo en desuso.

Para obtener más información, consulte [Administrar reglas de pertenencia dinámica](../enterprise-users/groups-dynamic-membership.md).

---

### <a name="new-microsoft-graph-app-permission-available-for-use-with-access-reviews"></a>Nuevo permiso de aplicación Microsoft Graph disponible para su uso con las revisiones de acceso

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Revisiones de acceso Identity Governance

Hemos introducido un nuevo permiso de aplicación Microsoft Graph, `AccessReview.ReadWrite.Membership`, que permite a las aplicaciones crear y recuperar automáticamente revisiones de acceso para pertenencias a grupos y asignaciones de aplicaciones. Este permiso se puede usar en los trabajos programados o como parte de la automatización, sin necesidad de un contexto de usuario que haya iniciado sesión.

Para obtener más información, consulte el [Ejemplo de cómo crear revisiones de acceso de Azure AD mediante los permisos de la aplicación Microsoft Graph con el blog de PowerShell ](https://techcommunity.microsoft.com/t5/Azure-Active-Directory/Example-how-to-create-Azure-AD-access-reviews-using-Microsoft/m-p/807241).

---

### <a name="azure-ad-activity-logs-are-now-available-for-government-cloud-instances-in-azure-monitor"></a>Los registros de actividad de Azure AD ahora están disponibles para las instancias de la nube de administración pública en Azure Monitor

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Nos complace anunciar que los registros de actividad de Azure AD ahora están disponibles para las instancias de la nube de administración pública en Azure Monitor Ahora puede enviar registros de Azure AD a su cuenta de almacenamiento o a un centro de eventos para integrarlos con las herramientas SIEM, como [Sumologic](../reports-monitoring/howto-integrate-activity-logs-with-sumologic.md), [Splunk](../reports-monitoring/howto-integrate-activity-logs-with-splunk.md) y [Arcsight](../reports-monitoring/howto-integrate-activity-logs-with-arcsight.md).

Para obtener más información sobre la configuración de Azure Monitor, consulte [Registros de actividad de Azure AD en Azure Monitor](../reports-monitoring/concept-activity-logs-azure-monitor.md#cost-considerations).

---

### <a name="update-your-users-to-the-new-enhanced-security-info-experience"></a>Actualice a sus usuarios a la nueva experiencia de información de seguridad mejorada

**Tipo:** **Categoría del servicio:** Característica modificada  **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

El 25 de septiembre de 2019, desactivaremos la antigua experiencia de información de seguridad no mejorada para registrar y administrar la información de seguridad del usuario y solo activaremos la nueva [versión mejorada](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Cool-enhancements-to-the-Azure-AD-combined-MFA-and-password/ba-p/354271). Esto significa que los usuarios ya no podrán usar la experiencia antigua.

Para más información sobre la experiencia de información de seguridad mejorada, consulte nuestra [documentación de administración](../authentication/concept-registration-mfa-sspr-combined.md) y nuestra [documentación de usuario](../user-help/security-info-setup-signin.md).

#### <a name="to-turn-on-this-new-experience-you-must"></a>Para activar esta nueva experiencia, debe:

1. Inicie sesión en Azure Portal como administrador global o administrador de usuarios.

2. Vaya a **Azure Active Directory > Configuración de usuario > Administrar la configuración de las características en versión preliminar del panel de acceso**.

3. En el área **Mejoras para los usuarios a la hora de utilizar las características en versión preliminar para registrar y administrar la información de seguridad**, seleccione **Seleccionado** y luego elija un grupo de usuarios o elija **Todos** a fin de activar esta función para todos los usuarios del inquilino.

4. En el **Los usuarios pueden usar las características de vista previa para registrar y administrar el área de información **de seguridad**,** seleccione **Ninguno**.

5. Guarde la configuración.

    Después de guardar la configuración, ya no tendrá acceso a la antigua experiencia de información de seguridad.

>[!Important]
>Si no completa estos pasos antes del 25 de septiembre de 2019, el inquilino de Azure Active Directory se habilitará automáticamente para mejorar la experiencia. Si le queda alguna duda, póngase en contacto con nosotros registrationpreview@microsoft.com.

---

### <a name="authentication-requests-using-post-logins-will-be-more-strictly-validated"></a>Las solicitudes de autenticación que utilizan inicios de sesión POST serán validadas de forma más rigurosa

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Estándares

A partir del 2 de septiembre de 2019, las solicitudes de autenticación que usan el método POST se validarán de forma más rigurosa con los estándares HTTP. Concretamente, los espacios y las comillas dobles (") ya no se quitarán de los valores del formulario de solicitud. No se espera que estos cambios interrumpan ningún cliente existente y ayudarán a garantizar que las solicitudes enviadas a Azure AD se controlan de forma confiable en todo momento.

Para obtener más información, consulte [Notificaciones de cambios de última hora de Azure AD](../develop/reference-breaking-changes.md#post-form-semantics-will-be-enforced-more-strictly---spaces-and-quotes-will-be-ignored).

---

## <a name="july-2019"></a>Julio de 2019

### <a name="plan-for-change-application-proxy-service-update-to-support-only-tls-12"></a>Planeado su cambio: actualización del servicio Proxy de la aplicación para que admita solo TLS 1.2

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Como ayuda para que le proporcionemos nuestro cifrado más seguro, vamos a empezar a limitar el acceso del servicio Proxy de la aplicación para solo a los protocolos TLS 1.2. Esta limitación se distribuirá inicialmente a los clientes que ya usen protocolos TLS 1,2, por lo que no verá el impacto. Los protocolos TLS 1.0 y TLS 1.1 quedarán totalmente en desuso el 31 de agosto de 2019. Los clientes que aún usen TLS 1.0 y TLS 1.1 recibirán un aviso con tiempo suficiente para prepararse para este cambio.

Para mantener la conexión con el servicio Proxy de la aplicación durante todo este cambio, es aconsejable asegurarse de que las combinaciones de cliente-servidor y explorador web-servidor se actualizan para que usen TLS 1.2. También es aconsejable asegurarse de incluir los sistemas cliente que usan los empleados para acceder a las aplicaciones publicadas mediante el servicio Proxy de la aplicación.

Para obtener más información, vea [Adición de una aplicación local para el acceso remoto mediante el proxy de aplicación en Azure Active Directory](../app-proxy/application-proxy-add-on-premises-application.md).

---

### <a name="plan-for-change-design-updates-are-coming-for-the-application-gallery"></a>Planeado su cambio: hay actualizaciones de diseño disponibles para la Galería de aplicaciones

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Aplicaciones empresariales SSO

Se van a realizar nuevos cambios en la interfaz de usuario en el diseño del área **Agregar desde la galería** de la hoja **Agregar una aplicación**. Estos cambios le ayudarán a encontrar más fácilmente las aplicaciones que admiten el aprovisionamiento automático, OpenID Connect, Lenguaje de marcado de aserción de seguridad (SAML) y el inicio de sesión único (SSO) con contraseña.

---

### <a name="plan-for-change-removal-of-the-mfa-server-ip-address-from-the-office-365-ip-address"></a>Planeado su cambio: Eliminación de la dirección IP del servidor MFA de la dirección IP de Office 365

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** MFA Seguridad y protección de la identidad

Vamos a quitar la dirección IP del servidor MFA del [servicio web de URL y dirección IP de Office 365](/office365/enterprise/office-365-ip-web-service). Si actualmente utiliza estas páginas para actualizar la configuración del firewall, debe asegurarse de incluir también la lista de direcciones IP documentadas en la sección **requisitos del firewall del servidor Azure Multi-Factor Authentication** del artículo [Introducción a Servidor Azure Multi-Factor Authentication](../authentication/howto-mfaserver-deploy.md#azure-multi-factor-authentication-server-firewall-requirements).

---

### <a name="app-only-tokens-now-require-the-client-app-to-exist-in-the-resource-tenant"></a>Los tokens que solo sean de aplicación ahora requieren que la aplicación cliente exista en el inquilino de recursos

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

El 26 de julio de 2019 cambiamos la forma en que proporcionamos tokens solo de aplicaciones a través [de la concesión de credenciales de cliente](../azuread-dev/v1-oauth2-client-creds-grant-flow.md). Anteriormente, las aplicaciones podían obtener tokens para llamar a otras aplicaciones, independientemente de si la aplicación cliente se encontraba en el inquilino. Hemos actualizado este comportamiento para que a los recursos de inquilino único, a veces denominados API Web, solo puedan llamarlos las aplicaciones cliente que existen en el inquilino de recursos.

Si la aplicación no se encuentra en el inquilino de recursos, recibirá el siguiente mensaje de error: `The service principal named <app_name> was not found in the tenant named <tenant_name>. This can happen if the application has not been installed by the administrator of the tenant.` Para solucionar este problema, debe crear la entidad de servicio de la aplicación cliente en el inquilino, para lo que debe usar el [punto de conexión de consentimiento del administrador](../develop/v2-permissions-and-consent.md#using-the-admin-consent-endpoint) o [mediante PowerShell](../develop/howto-authenticate-service-principal-powershell.md), lo que garantiza que el inquilino ha dado a la aplicación permiso para operar dentro del inquilino.

Para más información, consulte [Novedades en la autenticación](../develop/reference-breaking-changes.md#app-only-tokens-for-single-tenant-applications-are-only-issued-if-the-client-app-exists-in-the-resource-tenant).

> [!NOTE]
> El consentimiento existente entre el cliente y la API sigue sin ser necesario. Las aplicaciones aún deben realizar sus propias comprobaciones para saber si tienen autorización.

---

### <a name="new-passwordless-sign-in-to-azure-ad-using-fido2-security-keys"></a>Nuevo inicio de sesión sin contraseña para Azure AD mediante claves de seguridad FIDO2

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Los clientes de Azure AD ahora pueden establecer directivas para administrar las claves de seguridad FIDO2 para los usuarios y grupos de su organización. Los usuarios finales también pueden registrar automáticamente sus claves de seguridad, usar las claves para iniciar sesión con sus cuentas Microsoft en sitios web mientras estén en dispositivos compatibles con FIDO, así como iniciar sesión en sus dispositivos de Windows 10 unidos a Azure AD.

Para más información, consulte [Habilitación del inicio de sesión sin contraseña para Azure AD (versión preliminar)](../authentication/concept-authentication-passwordless.md) para obtener información relacionada con el administrador y [Configuración de la información de seguridad para usar una clave de seguridad (versión preliminar)](../user-help/security-info-setup-security-key.md) para la información relacionada con el usuario final.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---july-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (julio de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En julio de 2019, hemos agregado estas 18 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[Ungerboeck Software](../saas-apps/ungerboeck-software-tutorial.md), [Bright Pattern Omnichannel Contact Center](../saas-apps/bright-pattern-omnichannel-contact-center-tutorial.md), [Clever Nelly](../saas-apps/clever-nelly-tutorial.md), [AcquireIO](../saas-apps/acquireio-tutorial.md), [Looop](https://www.looop.co/schedule-a-demo/), [productboard](../saas-apps/productboard-tutorial.md), [MS Azure SSO Access for Ethidex Compliance Office&trade;](../saas-apps/ms-azure-sso-access-for-ethidex-compliance-office-tutorial.md), [Hype](../saas-apps/hype-tutorial.md), [Abstract](../saas-apps/abstract-tutorial.md), [Ascentis](../saas-apps/ascentis-tutorial.md), [Flipsnack](https://www.flipsnack.com/accounts/sign-in-sso.html), [Wandera](../saas-apps/wandera-tutorial.md), [TwineSocial](https://twinesocial.com/), [Kallidus](../saas-apps/kallidus-tutorial.md), [HyperAnna](../saas-apps/hyperanna-tutorial.md), [PharmID WasteWitness](https://pharmid.com/), [i2B Connect](https://www.i2b-online.com/sign-up-to-use-i2b-connect-here-sso-access/), [JFrog Artifactory](../saas-apps/jfrog-artifactory-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatización del aprovisionamiento de cuentas de usuario para estas aplicaciones SaaS recién admitidas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Supervisión e informes

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Dialpad](../saas-apps/dialpad-provisioning-tutorial.md)

- [Federated Directory](../saas-apps/federated-directory-provisioning-tutorial.md)

- [Figma](../saas-apps/figma-provisioning-tutorial.md)

- [Leapsome](../saas-apps/leapsome-provisioning-tutorial.md)

- [Peakon](../saas-apps/peakon-provisioning-tutorial.md)

- [Smartsheet](../saas-apps/smartsheet-provisioning-tutorial.md)

Para obtener más información sobre cómo proteger mejor la organización mediante el aprovisionamiento de cuentas de usuario de forma automatizada, vea [Automatización del aprovisionamiento de usuarios para aplicaciones SaaS con Azure AD](../app-provisioning/user-provisioning.md).

---

### <a name="new-azure-ad-domain-services-service-tag-for-network-security-group"></a>Nueva etiqueta de servicio de Azure AD Domain Services para el grupo de seguridad de red

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Si está cansado de administrar largas listas de rangos y direcciones IP, puede usar la nueva etiqueta de servicio de red **AzureActiveDirectoryDomainServices** en el grupo de seguridad de red de Azure para ayudar a proteger el tráfico que entra en la subred de la red virtual de Azure AD Domain Services.

Para más información acerca de esta nueva etiqueta de servicio, consulte [Grupos de seguridad de red para Azure AD Domain Services](../../active-directory-domain-services/network-considerations.md#network-security-groups-and-required-ports).

---

### <a name="new-security-audits-for-azure-ad-domain-services-public-preview"></a>Nuevas auditorías de seguridad para Azure AD Domain Services (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Nos complace anunciar el lanzamiento de la auditoría de seguridad de Azure AD Domain Services en versión preliminar pública. La auditoría de seguridad le ayuda a proporcionar información crítica acerca de los servicios de autenticación mediante la transmisión en secuencias de eventos de auditoría de seguridad a recursos de destino, entre los que se incluyen Azure Storage, áreas de trabajo de Azure Log Analytics y Azure Event Hubs, mediante el portal de Azure AD Domain Services.

Para más información, consulte [Habilitación de auditorías de seguridad para Azure AD Domain Services (versión preliminar)](../../active-directory-domain-services/security-audit-events.md).

---

### <a name="new-authentication-methods-usage--insights-public-preview"></a>Conclusiones y uso de los nuevos métodos de autenticación (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Supervisión e informes

Los informes de las conclusiones y uso de los nuevos métodos de autenticación pueden ayudarle a entender cómo se registran y usan características como Azure AD Multi-Factor Authentication y el autoservicio de restablecimiento de contraseña en su organización, lo que incluye el número de usuarios registrados para cada característica, la frecuencia con que se usa el autoservicio de restablecimiento de contraseña para restablecer contraseñas y el método de restablecimiento.

Para más información, consulte [Conclusiones y uso de los métodos de autenticación (versión preliminar)](../authentication/howto-authentication-methods-activity.md).

---

### <a name="new-security-reports-are-available-for-all-azure-ad-administrators-public-preview"></a>Hay nuevos informes de seguridad disponibles para todos los administradores de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

Ahora, todos los administradores de Azure ad pueden seleccionar el banner de la parte superior de los informes de seguridad existentes, como el informe **Usuarios marcados en riesgo**, para empezar a usar la nueva experiencia de seguridad, como se muestra en los informes **Usuarios de riesgo**  e **Inicios de sesión de riesgo**. Con el tiempo, todos los informes de seguridad pasarán de las versiones anteriores a las nuevas y los informes nuevos le proporcionarán las siguientes funcionalidades adicionales:

- Filtrado y ordenación avanzados

- Acciones en masa, como descartar el riesgo del usuario

- Confirmación de entidades en peligro o seguras

- Estado de riesgo, que abarca: En riesgo, Descartado, Corregido y Confirmado (en peligro)

Para más información, consulte [Informe de usuarios de riesgo](../identity-protection/howto-identity-protection-investigate-risk.md#risky-users) e [Informe de inicios de sesión peligrosos](../identity-protection/howto-identity-protection-investigate-risk.md#risky-sign-ins).

---

### <a name="new-security-audits-for-azure-ad-domain-services-public-preview"></a>Nuevas auditorías de seguridad para Azure AD Domain Services (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Nos complace anunciar el lanzamiento de la auditoría de seguridad de Azure AD Domain Services en versión preliminar pública. La auditoría de seguridad le ayuda a proporcionar información crítica acerca de los servicios de autenticación mediante la transmisión en secuencias de eventos de auditoría de seguridad a recursos de destino, entre los que se incluyen Azure Storage, áreas de trabajo de Azure Log Analytics y Azure Event Hubs, mediante el portal de Azure AD Domain Services.

Para más información, consulte [Habilitación de auditorías de seguridad para Azure AD Domain Services (versión preliminar)](../../active-directory-domain-services/security-audit-events.md).

---

### <a name="new-b2b-direct-federation-using-samlws-fed-public-preview"></a>Nueva federación directa B2B mediante SAML/WS-FED (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C

La federación directa facilita el trabajo con aquellos con asociados cuya solución de identidad administrada de tecnologías de la información no es Azure AD, ya que se trabaja con sistemas de identidad que admiten los estándares SAML o WS-Fed. Después de configurar una relación de federación directa con un asociado, todos los usuarios a los que invite desde dicho dominio pueden colaborar con usted mediante su cuenta de organización existente, lo que hace que la experiencia de sus invitados sea más fluida.

Para más información, consulte [Federación directa con AD FS y proveedores de terceros para usuarios invitados (versión preliminar)](../external-identities/direct-federation.md).

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatización del aprovisionamiento de cuentas de usuario para estas aplicaciones SaaS recién admitidas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Supervisión e informes

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Dialpad](../saas-apps/dialpad-provisioning-tutorial.md)

- [Federated Directory](../saas-apps/federated-directory-provisioning-tutorial.md)

- [Figma](../saas-apps/figma-provisioning-tutorial.md)

- [Leapsome](../saas-apps/leapsome-provisioning-tutorial.md)

- [Peakon](../saas-apps/peakon-provisioning-tutorial.md)

- [Smartsheet](../saas-apps/smartsheet-provisioning-tutorial.md)

Para más información acerca de cómo proteger mejor una organización mediante el aprovisionamiento automatizado de cuentas de usuario, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

---

### <a name="new-check-for-duplicate-group-names-in-the-azure-ad-portal"></a>Nueva comprobación de nombres de grupos duplicados en el portal de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Ahora, al crear o actualizar un nombre de grupo desde el portal de Azure AD, realizaremos una comprobación para ver si está duplicando un nombre de grupo existente en el recurso. Si determinamos que el nombre ya lo está usando otro grupo, se le pedirá que lo modifique.

Para más información, consulte [Administración de grupos en el portal de Azure AD](./active-directory-groups-create-azure-portal.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context).

---

### <a name="azure-ad-now-supports-static-query-parameters-in-reply-redirect-uris"></a>Azure AD ahora admite parámetros de consulta estáticos en las direcciones URI de respuesta (redireccionamiento)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Las aplicaciones de Azure AD ahora pueden registrar y usar identificadores URI (redireccionamiento) con parámetros de consulta estáticos (por ejemplo, `https://contoso.com/oauth2?idp=microsoft`) para las solicitudes de OAuth 2.0. El parámetro de consulta estático está sujeto a la coincidencia de cadenas en los identificadores URI de respuesta, al igual que las restantes partes del identificador URI. Si no hay ninguna cadena registrada que coincida con el identificador URI de redireccionamiento con la URL descodificada, se rechazará la solicitud. Si se encuentra el identificador URI, se usa toda la cadena para redirigir al usuario, incluido el parámetro de consulta estático.

Los identificadores URI de redireccionamiento dinámico siguen estando prohibidos, ya que representan un riesgo para la seguridad y no se pueden usar para conservar la información de estado a través de una solicitud de autenticación. Para este propósito, use el parámetro `state`.

Actualmente, las pantallas de registro de aplicaciones del Azure Portal aún bloquean los parámetros de consulta. Sin embargo, puede editar manualmente el manifiesto de la aplicación para agregar y probar los parámetros de consulta en su aplicación. Para más información, consulte [Novedades en la autenticación](../develop/reference-breaking-changes.md#redirect-uris-can-now-contain-query-string-parameters).

---

### <a name="activity-logs-ms-graph-apis-for-azure-ad-are-now-available-through-powershell-cmdlets"></a>Los registros de actividad (MS Graph API) para Azure AD ahora están disponibles a través de los cmdlets de PowerShell

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Nos complace anunciar que los registros de actividad de Azure AD (informes de auditoría e inicio de sesión) ahora están disponibles a través del módulo de Azure AD PowerShell. Antes podía crear sus propios scripts mediante los de MS Graph API y ahora hemos ampliado esa funcionalidad a los cmdlets de PowerShell.

Para más información acerca de cómo usar estos cmdlets, consulte [Cmdlets de PowerShell de Azure AD para informes](../reports-monitoring/reference-powershell-reporting.md).

---

### <a name="updated-filter-controls-for-audit-and-sign-in-logs-in-azure-ad"></a>Controles de filtro actualizados para los registros de auditoría e inicio de sesión en Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Hemos actualizado los informes de auditoría e inicio de sesión, por lo que ahora puede aplicar varios filtros sin tener que agregarlos como columnas en las pantallas de informes. Además, ahora puede decidir el número de filtros que desea mostrar en la pantalla. Estas actualizaciones funcionan conjuntamente para que los informes sean más fáciles de leer y se ajusten más a sus necesidades.

Para más información acerca de estas actualizaciones, consulte [Filtrado de registros de auditoría](../reports-monitoring/concept-audit-logs.md#filtering-audit-logs) y [Filtrado de las actividades de inicio de sesión](../reports-monitoring/concept-sign-ins.md#filter-sign-in-activities)de actividades de inicio de sesión.

---

## <a name="june-2019"></a>Junio de 2019

### <a name="new-riskdetections-api-for-microsoft-graph-public-preview"></a>Nueva API riskDetections de Microsoft Graph (versión preliminar)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

Nos complace anunciar que la nueva API riskDetections de Microsoft Graph ya se encuentra en fase de versión preliminar pública. Puede usar esta nueva API para ver una lista de detecciones de riesgos de inicio de sesión y usuario relativos a Identity Protection de su organización. También puede utilizarla para consultar más eficazmente las detecciones de riesgos, incluidos detalles sobre el tipo de detección, el estado, el nivel y mucho más.

Para obtener más información, vea la [documentación de referencia de la API de detección de riesgos](/graph/api/resources/riskdetection?view=graph-rest-beta&preserve-view=true).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---june-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (junio de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En junio de 2019, hemos agregado estas 22 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[Azure AD SAML Toolkit](../saas-apps/saml-toolkit-tutorial.md), [Otsuka Shokai (大塚商会)](../saas-apps/otsuka-shokai-tutorial.md), [ANAQUA](../saas-apps/anaqua-tutorial.md), [Azure VPN Client](https://portal.azure.com/), [ExpenseIn](../saas-apps/expensein-tutorial.md), [Helper Helper](../saas-apps/helper-helper-tutorial.md), [Costpoint](../saas-apps/costpoint-tutorial.md), [GlobalOne](../saas-apps/globalone-tutorial.md), [Mercedes-Benz In-Car Office](https://me.secure.mercedes-benz.com/), [Skore](https://app.justskore.it/), [Oracle Cloud Infrastructure Console](../saas-apps/oracle-cloud-tutorial.md), [CyberArk SAML Authentication](../saas-apps/cyberark-saml-authentication-tutorial.md), [Scrible Edu](https://www.scrible.com/sign-in/#/create-account), [PandaDoc](../saas-apps/pandadoc-tutorial.md), [Perceptyx](https://apexdata.azurewebsites.net/docs.microsoft.com/azure/active-directory/saas-apps/perceptyx-tutorial), [Proptimise OS](https://proptimise.co.uk/), [Vtiger CRM (SAML)](../saas-apps/vtiger-crm-saml-tutorial.md), Oracle Access Manager for Oracle Retail Merchandising, Oracle Access Manager for Oracle E-Business Suite, Oracle IDCS for E-Business Suite, Oracle IDCS for PeopleSoft y Oracle IDCS for JD Edwards

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatización del aprovisionamiento de cuentas de usuario para estas aplicaciones SaaS recién admitidas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Supervisión e informes

Ahora, puede automatizar la creación, actualización y eliminación de cuentas de usuario para estas aplicaciones recién integradas:

- [Zoom](../saas-apps/zoom-provisioning-tutorial.md)

- [Envoy](../saas-apps/envoy-provisioning-tutorial.md)

- [Proxyclick](../saas-apps/proxyclick-provisioning-tutorial.md)

- [4me](../saas-apps/4me-provisioning-tutorial.md)

Para obtener más información sobre cómo proteger mejor la organización mediante el aprovisionamiento de cuentas de usuario de forma automatizada, vea [Automatización del aprovisionamiento de usuarios para aplicaciones SaaS con Azure AD](../app-provisioning/user-provisioning.md).

---

### <a name="view-the-real-time-progress-of-the-azure-ad-provisioning-service"></a>Consulta del progreso en tiempo real del servicio de aprovisionamiento de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Administración del ciclo de vida de la identidad

Hemos actualizado la experiencia de aprovisionamiento de Azure AD para incluir una nueva barra de progreso que muestra a qué altura está el proceso de aprovisionamiento de usuario. Esta experiencia mejorada también proporciona información sobre el número de usuarios aprovisionados durante el ciclo actual, así como los usuarios aprovisionados hasta la fecha.

Para obtener más información, vea [Comprobación del estado de aprovisionamiento](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md).

---

### <a name="company-branding-now-appears-on-sign-out-and-error-screens"></a>Ahora, la personalización de marca aparece en las pantallas de cierre de sesión y de error

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Hemos actualizado Azure AD para que, ahora, la personalización de marca de su empresa aparezca en las pantallas de cierre de sesión y de error, así como en la página de inicio de sesión. Para activar esta característica no es preciso hacer nada, Azure AD usa los recursos que ya están configurados en el área **Personalización de marca de empresa** de Azure Portal.

Para obtener más información sobre cómo configurar la personalización de marca de su empresa, vea [Incorporación de la personalización de marca en las páginas de Azure Active Directory de la organización](./customize-branding.md).

---

### <a name="azure-multi-factor-authentication-mfa-server-is-no-longer-available-for-new-deployments"></a>El Servidor Azure Multi-Factor Authentication (MFA) ya no está disponible para nuevas implementaciones

**Tipo:** **Categoría del servicio:** En desuso **Funcionalidad del producto:** MFA Seguridad y protección de la identidad

A partir del 1 de julio de 2019, Microsoft ya no ofrecerá el Servidor MFA para implementaciones nuevas. Ahora, los clientes nuevos que quieran exigir la autenticación multifactor en su organización deberán usar Azure AD Multi-factor Authentication basado en la nube. Los clientes que tengan activado el Servidor MFA desde antes del 1 de julio no apreciarán ningún cambio. Podrán seguir descargando las versiones más recientes, obteniendo actualizaciones futuras y generando credenciales de activación.

Para obtener más información, vea [Introducción al Servidor Azure Multi-Factor Authentication](../authentication/howto-mfaserver-deploy.md). Para obtener más información sobre Azure AD Multi-Factor Authentication basado en la nube, vea [Planeamiento de una implementación de Azure AD Multi-Factor Authentication basado en la nube](../authentication/howto-mfa-getstarted.md).

---

## <a name="may-2019"></a>Mayo de 2019

### <a name="service-change-future-support-for-only-tls-12-protocols-on-the-application-proxy-service"></a>Cambio de servicio: próxima compatibilidad únicamente con protocolos TLS 1.2 en el servicio Application Proxy

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

A fin de proporcionar el mejor cifrado a nuestros clientes, estamos limitando el acceso a únicamente protocolos TLS 1.2 en el servicio Application Proxy. Este cambio se está implantando gradualmente en clientes que ya usan protocolos TLS 1.2 exclusivamente, por lo que no deberían apreciar cambio alguno.

La obsolescencia de TLS 1.0 y TLS 1.1 tendrá lugar el 31 de agosto de 2019, pero avisaremos con antelación para que tenga tiempo para prepararse para este cambio. Para prepararse para este cambio, asegúrese de que sus combinaciones de servidor cliente y servidor de examinador (incluidos los clientes que los usuarios usan para tener acceso a aplicaciones publicadas a través de Application Proxy) se actualizan para usar el protocolo TLS 1.2 y, así, mantengan la conexión con el servicio Application Proxy. Para obtener más información, vea [Adición de una aplicación local para el acceso remoto mediante el proxy de aplicación en Azure Active Directory](../app-proxy/application-proxy-add-on-premises-application.md#prerequisites).

---

### <a name="use-the-usage-and-insights-report-to-view-your-app-related-sign-in-data"></a>Utilización del informe de uso y conclusiones para ver datos de inicio de sesión relativos a las aplicaciones

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Supervisión e informes

Ahora, puede usar el informe de uso y conclusiones, ubicado en el área **Aplicaciones empresariales** de Azure Portal, para obtener una vista de los datos de inicio de sesión relativos a las aplicaciones, con información sobre lo siguiente:

- Aplicaciones más usadas en la organización

- Aplicaciones con los inicios de sesión con más errores

- Principales errores de inicio de sesión de cada aplicación

Para obtener más información sobre esta característica, vea [Informe de uso y conclusiones en el portal de Azure Active Directory](../reports-monitoring/concept-usage-insights-report.md).

---

### <a name="automate-your-user-provisioning-to-cloud-apps-using-azure-ad"></a>Automatización del aprovisionamiento de usuarios a aplicaciones en la nube mediante Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Supervisión e informes

Siga estos nuevos tutoriales para utilizar el servicio de aprovisionamiento de Azure AD para automatizar la creación, eliminación y actualización de cuentas de usuario de las siguientes aplicaciones en la nube:

- [Comeet](../saas-apps/comeet-recruiting-software-provisioning-tutorial.md)

- [DynamicSignal](../saas-apps/dynamic-signal-provisioning-tutorial.md)

- [KeeperSecurity](../saas-apps/keeper-password-manager-digitalvault-provisioning-tutorial.md)

También puede seguir este nuevo [tutorial de Dropbox](../saas-apps/dropboxforbusiness-provisioning-tutorial.md), que proporciona información sobre cómo aprovisionar objetos de grupo.

Para obtener más información sobre cómo proteger mejor la organización a través del aprovisionamiento automatizado de cuentas de usuario, vea [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

---

### <a name="identity-secure-score-is-now-available-in-azure-ad-general-availability"></a>La puntuación segura de identidad ya está disponible en Azure AD (disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** N/D Seguridad y protección de la identidad

Ahora, puede supervisar y mejorar su posición de seguridad de identidad gracias a la característica de puntuación segura de identidad de Azure AD. La característica de puntuación segura de identidad utiliza un mismo panel para ayudarle a:

- Medir de forma objetiva su nivel de seguridad de la identidad, según una puntuación de entre 1 y 223.

- Planear la realización de mejoras en la seguridad de la identidad.

- Ver si las mejoras han logrado sus objetivos de seguridad.

Para obtener más información sobre la característica de puntuación segura de identidad, vea [¿Qué es la puntuación segura de identidad en Azure Active Directory?](./identity-secure-score.md)

---

### <a name="new-app-registrations-experience-is-now-available-general-availability"></a>Nueva experiencia Registros de aplicaciones ya disponible (disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Experiencia para el desarrollador

La nueva experiencia [Registros de aplicaciones](https://aka.ms/appregistrations) ya está disponible en versión de disponibilidad general. Esta nueva experiencia incluye todas las características clave a las que está acostumbrado de Azure Portal y del portal de Registros de aplicaciones, y las mejora a través de lo siguiente:

- **Una mejor administración de las aplicaciones.** En lugar de ver las aplicaciones en diversos portales, ahora puede verlas todas en una misma ubicación.

- **Un registro de aplicaciones simplificado.** Desde la experiencia de navegación mejorada a la experiencia de selección de permisos renovada, ahora es más fácil registrar y administrar sus aplicaciones.

- **Una información más pormenorizada.** Encontrará más detalles sobre la aplicación, incluidas guías de inicio rápido y otras muchas cosas.

Para obtener más información, vea [Plataforma de identidad de Microsoft](../develop/index.yml) y la entrada de blog que anuncia que la [experiencia Registros de aplicación ya está disponible de forma general](https://developer.microsoft.com/identity/blogs/new-app-registrations-experience-is-now-generally-available/) .

---

### <a name="new-capabilities-available-in-the-risky-users-api-for-identity-protection"></a>Nuevas capacidades disponibles en la API correspondiente a los usuarios de riesgo de Identity Protection

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

Nos complace anunciar que, ahora, puede usar la API correspondiente a los usuarios de riesgo para recuperar el historial de riesgos de los usuarios, descartar usuarios de riesgo y confirmar usuarios como usuarios de riesgo. Este cambio ayuda a actualizar el estado de riesgo de los usuarios de forma más eficaz y a entender sus historiales de riesgos.

Para obtener más información, vea la [documentación de referencia de la API correspondiente a los usuarios de riesgo](/graph/api/resources/riskyuser?view=graph-rest-beta&preserve-view=true).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---may-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (mayo de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En mayo de 2019, hemos agregado estas 21 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[Freedcamp](../saas-apps/freedcamp-tutorial.md), [Real Links](../saas-apps/real-links-tutorial.md), [Kianda](https://app.kianda.com/sso/OpenID/AzureAD/), [Simple Sign](../saas-apps/simple-sign-tutorial.md), [Braze](../saas-apps/braze-tutorial.md), [Displayr](../saas-apps/displayr-tutorial.md), [Templafy](../saas-apps/templafy-tutorial.md), [Marketo Sales Engage](https://toutapp.com/login), [ACLP](../saas-apps/aclp-tutorial.md), [OutSystems](../saas-apps/outsystems-tutorial.md), [Meta4 Global HR](../saas-apps/meta4-global-hr-tutorial.md), [Quantum Workplace](../saas-apps/quantum-workplace-tutorial.md), [Cobalt](../saas-apps/cobalt-tutorial.md), [webMethods API Cloud](../saas-apps/webmethods-integration-cloud-tutorial.md), [RedFlag](https://pocketstop.com/redflag/), [Whatfix](../saas-apps/whatfix-tutorial.md), [Control](../saas-apps/control-tutorial.md), [JOBHUB](../saas-apps/jobhub-tutorial.md), [NEOGOV](../saas-apps/neogov-tutorial.md), [Foodee](../saas-apps/foodee-tutorial.md) y [MyVR](../saas-apps/myvr-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="improved-groups-creation-and-management-experiences-in-the-azure-ad-portal"></a>Experiencias mejoradas de administración y creación de grupos en el portal de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Hemos realizado mejoras en las experiencias relativas a grupos del portal de Azure AD. Estas mejoras permiten a los administradores administrar de mejor forma las listas de grupos y las listas de miembros, así como proporcionar más opciones de creación.

Estas mejoras incluyen:

- Filtrado básico por tipo de pertenencia y tipo de grupo.

- Incorporación de columnas nuevas, como Origen y Dirección de correo electrónico.

- Posibilidad de seleccionar varios grupos, miembros y listas de propietarios para eliminarlos con facilidad.

- Posibilidad de elegir una dirección de correo electrónico y de agregar propietarios durante la creación del grupo.

Para obtener más información vea [Creación de un grupo básico e incorporación de miembros con Azure Active Directory](./active-directory-groups-create-azure-portal.md).

---

### <a name="configure-a-naming-policy-for-office-365-groups-in-azure-ad-portal-general-availability"></a>Configuración de una directiva de nomenclatura para grupos de Office 365 en el portal de Azure AD (disponibilidad general)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Administración de grupos Colaboración

Ahora, los administradores pueden configurar una directiva de nomenclatura de grupos de Office 365 mediante el portal de Azure AD. Este cambio ayuda a aplicar las convenciones de nomenclatura coherentes para los grupos de Office 365 creados o editados por los usuarios de su organización.

La directiva de nomenclatura de grupos de Office 365 se puede configurar de dos maneras diferentes:

- Definir prefijos o sufijos, que se agregan automáticamente al nombre de un grupo.

- Cargar un conjunto personalizado de palabras bloqueadas de la organización que no se permiten en los nombres de grupo (por ejemplo, "CEO, nómina, RR. HH.").

Para obtener más información, vea [Aplicación de una directiva de nomenclatura en los grupos de Office 365](../enterprise-users/groups-naming-policy.md).

---

### <a name="microsoft-graph-api-endpoints-are-now-available-for-azure-ad-activity-logs-general-availability"></a>Ahora, los puntos de conexión de la API de Microsoft Graph están disponibles para los registros de actividad de Azure AD (disponibilidad general)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Nos complace anunciar la disponibilidad general de compatibilidad de puntos de conexión de la API de Microsoft Graph en los registros de actividad de Azure AD. Con este lanzamiento, ahora puede usar la versión 1.0 tanto de los registros de auditoría de Azure AD como de las API de registros de inicio de sesión.

Para obtener más información, vea [Información general sobre la API de registros de auditoría de Azure AD](/graph/api/resources/azure-ad-auditlog-overview).

---

### <a name="administrators-can-now-use-conditional-access-for-the-combined-registration-process-public-preview"></a>Ahora, los administradores pueden usar el acceso condicional en el proceso de registro combinado (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

Ahora, los administradores pueden crear directivas de acceso condicional para su uso en la página de registro combinado. Esto incluye la puesta en marcha de directivas que permiten el registro bajo estos supuestos:

- Los usuarios están en una red de confianza.

- Los usuarios presentan un bajo riesgo de inicio de sesión.

- Los usuarios están en un dispositivo administrado.

- Los usuarios han aceptado las condiciones de uso de la organización.

Para obtener más información sobre el acceso condicional y el restablecimiento de contraseña, puede ver la entrada de blog sobre el [acceso condicional en la experiencia de registro combinado de restablecimiento de contraseña y MFA de Azure AD](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Conditional-access-for-the-Azure-AD-combined-MFA-and-password/ba-p/566348). Para obtener más información sobre las directivas de acceso condicional en el proceso de registro combinado, vea [Directivas de acceso condicional para el registro combinado](../authentication/howto-registration-mfa-sspr-combined.md#conditional-access-policies-for-combined-registration). Para obtener más información sobre la característica Condiciones de uso de Azure AD, vea [Característica Condiciones de uso de Azure Active Directory](../conditional-access/terms-of-use.md).

---

## <a name="april-2019"></a>Abril de 2019

### <a name="new-azure-ad-threat-intelligence-detection-is-now-available-as-part-of-azure-ad-identity-protection"></a>Nueva detección de la inteligencia sobre amenazas de Azure AD ya disponible como parte de Azure AD Identity Protection

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Identity Protection Seguridad y protección de la identidad

La detección de la inteligencia sobre amenazas de Azure AD ya está disponible como parte de la característica actualizada Azure AD Identity Protection. Esta nueva funcionalidad ayuda a identificar una actividad de usuario poco común para el usuario en cuestión o una actividad que encaja con patrones de ataque conocidos según los orígenes de la inteligencia sobre amenazas internas y externas de Microsoft.

Para obtener más información sobre la versión actualizada de Azure AD Identity Protection, vea la entrada de blog sobre las [cuatro grandes mejoras de Azure AD Identity Protection que ya están en fase de versión preliminar pública](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Four-major-Azure-AD-Identity-Protection-enhancements-are-now-in/ba-p/326935) y el artículo [¿Qué es Azure Active Directory Identity Protection (actualizado)?](../identity-protection/overview-identity-protection.md) . Para obtener más información sobre la detección de inteligencia sobre amenazas de Azure AD, vea el artículo [Referencia sobre eventos de riesgo de Azure Active Directory Identity Protection](../identity-protection/concept-identity-protection-risks.md).

---

### <a name="azure-ad-entitlement-management-is-now-available-public-preview"></a>La administración de derechos de Azure AD ya está disponible (versión preliminar)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Governance Identity Governance

La administración de derechos de Azure AD, ahora en versión preliminar pública, permite a los clientes delegar la administración de paquetes de acceso, que define cómo los empleados y los asociados comerciales pueden solicitar acceso, quién debe aprobar ese acceso y cuánto va a durar ese acceso. Los paquetes de acceso pueden administrar la pertenencia a grupos de Azure AD y Office 365, las asignaciones de roles en aplicaciones empresariales y las asignaciones de roles para sitios de SharePoint Online. Encontrará más información sobre la administración de derechos en el artículo de [información general sobre la administración de derechos de Azure AD](../governance/entitlement-management-overview.md). Para obtener más información sobre el amplio abanico de características de Azure AD Identity Governance, incluidos Privileged Identity Management, revisiones del acceso y las condiciones de uso, vea [¿Qué es Azure AD Identity Governance?](../governance/identity-governance-overview.md)

---

### <a name="configure-a-naming-policy-for-office-365-groups-in-azure-ad-portal-public-preview"></a>Configuración de una directiva de nomenclatura para grupos de Office 365 en el portal de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Ahora, los administradores pueden configurar una directiva de nomenclatura de grupos de Office 365 mediante el portal de Azure AD. Este cambio ayuda a aplicar las convenciones de nomenclatura coherentes para los grupos de Office 365 creados o editados por los usuarios de su organización.

La directiva de nomenclatura de grupos de Office 365 se puede configurar de dos maneras diferentes:

- Definir prefijos o sufijos, que se agregan automáticamente al nombre de un grupo.

- Cargar un conjunto personalizado de palabras bloqueadas de la organización que no se permiten en los nombres de grupo (por ejemplo, "CEO, nómina, RR. HH.").

Para obtener más información, vea [Aplicación de una directiva de nomenclatura en los grupos de Office 365](../enterprise-users/groups-naming-policy.md).

---

### <a name="azure-ad-activity-logs-are-now-available-in-azure-monitor-general-availability"></a>Los registros de actividad de Azure AD ya están disponibles en Azure Monitor (disponibilidad general)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Para que sea más fácil atender sus comentarios sobre las visualizaciones de los registros de actividad de Azure AD, hemos incluido una nueva característica Información en Log Analytics. Esta característica sirve para obtener información sobre los recursos de Azure AD mediante nuestras plantillas interactivas, llamadas libros. Estos libros pregenerados proporcionan detalles sobre las aplicaciones o los usuarios, como lo siguiente:

- **Inicios de sesión** Proporciona detalles sobre las aplicaciones y los usuarios, incluida la ubicación de inicio de sesión, el sistema operativo o el cliente y versión del explorador que se usa y el número de inicios de sesión correctos o con errores.

- **Autenticación heredada y acceso condicional** Proporciona detalles sobre las aplicaciones y los usuarios que usan la autenticación heredada, incluido el uso de Multi-Factor Authentication desencadenado por las directivas de acceso condicional, las aplicaciones que usan directivas de acceso condicional, etc.

- **Análisis de errores de inicio de sesión** Ayuda a averiguar si los errores de inicio de sesión se deben a una acción del usuario, a problemas de directiva o a la infraestructura existente.

- **Informes personalizados** Puede crear libros o editar los ya existentes para ayudar a personalizar la característica Información de su organización.

Para obtener más información, vea [Cómo usar los libros de Azure Monitor en informes de Azure Active Directory](../reports-monitoring/howto-use-azure-monitor-workbooks.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---april-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (abril de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En abril de 2019, hemos agregado estas 21 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[SAP Fiori](../saas-apps/sap-fiori-tutorial.md), [HRworks Single Sign-On](../saas-apps/hrworks-single-sign-on-tutorial.md), [Percolate](../saas-apps/percolate-tutorial.md), [MobiControl](../saas-apps/mobicontrol-tutorial.md), [Citrix NetScaler](../saas-apps/citrix-netscaler-tutorial.md), [Shibumi](../saas-apps/shibumi-tutorial.md), [Benchling](../saas-apps/benchling-tutorial.md), [MileIQ](https://mileiq.onelink.me/991934284/7e980085), [PageDNA](../saas-apps/pagedna-tutorial.md), [EduBrite LMS](../saas-apps/edubrite-lms-tutorial.md), [RStudio Connect](../saas-apps/rstudio-connect-tutorial.md), [AMMS](../saas-apps/amms-tutorial.md), [Mitel Connect](../saas-apps/mitel-connect-tutorial.md), [Alibaba Cloud (Role-based SSO)](../saas-apps/alibaba-cloud-service-role-based-sso-tutorial.md), [Certent Equity Management](../saas-apps/certent-equity-management-tutorial.md), [Sectigo Certificate Manager](../saas-apps/sectigo-certificate-manager-tutorial.md), [GreenOrbit](../saas-apps/greenorbit-tutorial.md), [Workgrid](../saas-apps/workgrid-tutorial.md), [monday.com](../saas-apps/mondaycom-tutorial.md), [SurveyMonkey Enterprise](../saas-apps/surveymonkey-enterprise-tutorial.md) e [Indiggo](https://indiggolead.com/)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-access-reviews-frequency-option-and-multiple-role-selection"></a>Nueva opción de frecuencia de revisiones de acceso y selección de varios roles

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Revisiones de acceso Identity Governance

Las nuevas actualizaciones de las revisiones de acceso de Azure AD permiten lo siguiente:

- Cambiar la frecuencia de las revisiones de acceso a **Semestral**, además de las opciones ya existentes (Semanal, Mensual, Trimestral y Anual).

- Seleccionar varios roles de recursos de Azure y de Azure AD al crear una revisión de acceso único. En esta situación, todos los roles se configuran igual y se notifica a todos los revisores al mismo tiempo.

Para obtener más información sobre cómo crear una revisión de acceso, vea [Creación de una revisión de acceso de los grupos o las aplicaciones en las revisiones de acceso de Azure AD](../governance/create-access-review.md).

---

### <a name="azure-ad-connect-email-alert-systems-are-transitioning-sending-new-email-sender-information-for-some-customers"></a>Transición de los sistemas de alertas por correo electrónico de Azure AD Connect en curso; se está enviando información de nuevo remitente de correo electrónico a algunos clientes

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Sincronización de AD Plataforma

Azure AD Connect está en fase de transición de nuestros sistemas de alertas de correo electrónico, lo que puede hacer que algunos clientes vean un nuevo remitente de correo electrónico. Para solucionar este problema, hay que agregar `azure-noreply@microsoft.com` a la lista de permitidos de la organización, o no podrá seguir recibiendo alertas importantes de los servicios de Office 365, Azure o de sincronización.

---

### <a name="upn-suffix-changes-are-now-successful-between-federated-domains-in-azure-ad-connect"></a>Los cambios en los sufijos UPN ya se realizan correctamente entre dominios federados en Azure AD Connect

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Sincronización de AD Plataforma

Ahora puede cambiar correctamente el sufijo UPN de un usuario de un dominio federado a otro en Azure AD Connect. Esta corrección implica que ya no debería aparecer el mensaje de error FederatedDomainChangeError durante el ciclo de sincronización, ni recibir una notificación por correo electrónico que informa de que el objeto no se puede actualizar en Azure Active Directory porque el atributo [FederatedUser.UserPrincipalName] no es válido y pide actualizar el valor en los servicios de directorio local.

Para obtener más información, vea [Solución de errores durante la sincronización](../hybrid/tshoot-connect-sync-errors.md#federateddomainchangeerror).

---

### <a name="increased-security-using-the-app-protection-based-conditional-access-policy-in-azure-ad-public-preview"></a>Mayor seguridad con la directiva de acceso condicional basada en la protección de aplicaciones en Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

Ahora, el acceso condicional basado en la protección de aplicaciones está disponible a través de la directiva **Requerir protección de aplicaciones**. Esta nueva directiva ayuda a aumentar la seguridad de la organización al tratar de impedir que:

- Los usuarios obtengan acceso a las aplicaciones sin contar con una licencia de Microsoft Intune.

- Los usuarios no puedan obtener una directiva de protección de aplicaciones de Microsoft Intune.

- Los usuarios obtengan acceso a las aplicaciones sin una directiva de protección de aplicaciones configurada de Microsoft Intune.

Para obtener más información, vea [Cómo usar la aplicación Requerir protección de aplicaciones para el acceso a aplicaciones en la nube mediante el acceso condicional](../conditional-access/app-protection-based-conditional-access.md).

---

### <a name="new-support-for-azure-ad-single-sign-on-and-conditional-access-in-microsoft-edge-public-preview"></a>Nueva compatibilidad con el acceso condicional y el inicio de sesión único de Azure AD en Microsoft Edge (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

Hemos mejorado la compatibilidad de Azure AD para Microsoft Edge, incluido proporcionar nueva compatibilidad con el inicio de sesión único en Azure AD y el acceso condicional. Si ya usó previamente Microsoft Intune Managed Browser, ahora puede usar Microsoft Edge en su lugar.

Para obtener más información sobre cómo configurar y administrar los dispositivos y aplicaciones mediante el acceso condicional, vea [Requerir dispositivos administrados para el acceso a aplicaciones en la nube con el acceso condicional](../conditional-access/require-managed-devices.md) y [Requerir aplicaciones de cliente aprobadas para el acceso a aplicaciones en la nube con el acceso condicional](../conditional-access/app-based-conditional-access.md). Para obtener más información sobre cómo administrar el acceso mediante Microsoft Edge con directivas de Microsoft Intune, vea [Administración del acceso web con un explorador protegido por directiva de Microsoft Intune](/intune/app-configuration-managed-browser).

---

## <a name="march-2019"></a>Marzo de 2019

### <a name="identity-experience-framework-and-custom-policy-support-in-azure-active-directory-b2c-is-now-available-ga"></a>Identity Experience Framework y las directivas personalizadas ya están disponibles en Azure Active Directory B2C (GA)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Ahora es posible crear directivas personalizadas en Azure AD B2C, incluidas las siguientes tareas, que se admiten a escala y bajo nuestro SLA de Azure:

- Crear y cargar trayectos de autenticación personalizada de los usuarios mediante directivas personalizadas.

- Describir los recorridos del usuario paso a paso como intercambios entre proveedores de notificaciones.

- Definir la creación de ramas condicional en recorridos del usuario.

- Transformar y asignar notificaciones para su uso en las comunicaciones y las decisiones en tiempo real.

- Usar servicios habilitados por la API de REST en los trayectos de autenticación personalizada de los usuarios. Por ejemplo, con proveedores de correo electrónico, CRM y sistemas de autorización propietarios.

- Federarse con proveedores de identidades que cumplen con el protocolo OpenIDConnect. Por ejemplo, con Azure AD multiinquilino, proveedores de cuentas de redes sociales o proveedores de verificación de dos factores.

Para más información sobre la creación de directivas personalizadas, vea [Notas para desarrolladores de directivas personalizadas en Azure Active Directory B2C](../../active-directory-b2c/custom-policy-developer-notes.md) y lea la [entrada de blog de Alex Simon, que incluye casos prácticos](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-custom-policies-to-build-your-own-identity-journeys/ba-p/382791).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---march-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (marzo de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En marzo de 2019, hemos agregado estas 14 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[ISEC7 Mobile Exchange Delegate](https://www.isec7.com/english/), [MediusFlow](https://office365.cloudapp.mediusflow.com/), [ePlatform](../saas-apps/eplatform-tutorial.md), [Fulcrum](../saas-apps/fulcrum-tutorial.md), [ExcelityGlobal](../saas-apps/excelityglobal-tutorial.md), [Explanation-Based Auditing System](../saas-apps/explanation-based-auditing-system-tutorial.md), [Lean](../saas-apps/lean-tutorial.md), [Powerschool Performance Matters](../saas-apps/powerschool-performance-matters-tutorial.md), [Cinode](https://cinode.com/), [Iris Intranet](../saas-apps/iris-intranet-tutorial.md), [Empactis](../saas-apps/empactis-tutorial.md), [SmartDraw](../saas-apps/smartdraw-tutorial.md), [Confirmit Horizons](../saas-apps/confirmit-horizons-tutorial.md) y [TAS](../saas-apps/tas-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-zscaler-and-atlassian-provisioning-connectors-in-the-azure-ad-gallery---march-2019"></a>Nuevos conectores de aprovisionamiento de Zscaler y Atlassian en la galería Azure AD (marzo de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Integración de terceros

Automatice la creación, la actualización y la eliminación de cuentas de usuario para las siguientes aplicaciones:

[Zscaler](../saas-apps/zscaler-provisioning-tutorial.md), [Zscaler Beta](../saas-apps/zscaler-beta-provisioning-tutorial.md), [Zscaler One](../saas-apps/zscaler-one-provisioning-tutorial.md), [Zscaler Two](../saas-apps/zscaler-two-provisioning-tutorial.md), [Zscaler Three](../saas-apps/zscaler-three-provisioning-tutorial.md), [Zscaler ZSCloud](../saas-apps/zscaler-zscloud-provisioning-tutorial.md) y [Atlassian Cloud](../saas-apps/atlassian-cloud-provisioning-tutorial.md)

Para obtener más información sobre cómo proteger mejor la organización a través del aprovisionamiento automatizado de cuentas de usuario, vea [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

---

### <a name="restore-and-manage-your-deleted-office-365-groups-in-the-azure-ad-portal"></a>Restaurar y administrar los grupos de Office 365 eliminados en el portal de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Ahora es posible ver y administrar los grupos de Office 365 eliminados desde el portal de Azure AD. Este cambio ayuda a ver qué grupos se pueden restaurar, además de permitir eliminar de forma permanente los grupos que la organización no necesite.

Para obtener más información, vea [Restauración de grupos expirados o eliminados](../enterprise-users/groups-restore-deleted.md#view-and-manage-the-deleted-microsoft-365-groups-that-are-available-to-restore).

---

### <a name="single-sign-on-is-now-available-for-azure-ad-saml-secured-on-premises-apps-through-application-proxy-public-preview"></a>El inicio de sesión único está ahora disponible para aplicaciones locales protegidas por SAML de Azure AD a través del proxy de la aplicación (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Ahora se puede proporcionar una experiencia de inicio de sesión único (SSO) en aplicaciones locales con autenticación SAML, junto con acceso remoto a estas aplicaciones a través del proxy de la aplicación. Para obtener más información sobre cómo configurar el SSO de SAML con las aplicaciones locales, vea [Inicio de sesión único de SAML para aplicaciones locales con proxy de aplicación (versión preliminar)](../manage-apps/application-proxy-configure-single-sign-on-on-premises-apps.md).

---

### <a name="client-apps-in-request-loops-will-be-interrupted-to-improve-reliability-and-user-experience"></a>Se interrumpirán las aplicaciones cliente en bucles de solicitud para mejorar la confiabilidad y la experiencia del usuario

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Las aplicaciones cliente pueden emitir incorrectamente cientos de solicitudes del mismo inicio de sesión durante un breve período de tiempo. Todas estas solicitudes, aprobadas o no, contribuyen a una experiencia de usuario deficiente y a mayores cargas de trabajo para el IDP, lo que aumenta la latencia de todos los usuarios y reduce la disponibilidad del IDP.

Esta actualización envía el error `invalid_grant`: `AADSTS50196: The server terminated an operation because it encountered a loop while processing a request` a las aplicaciones cliente que emiten solicitudes duplicadas varias veces en un breve período de tiempo, más allá del ámbito de operación normal. Las aplicaciones cliente que experimentan este problema deben mostrar un mensaje interactivo que requiera que el usuario vuelva a iniciar sesión. Para obtener más información sobre este cambio y sobre cómo corregir la aplicación si se encuentra con este error, vea [Novedades en la autenticación](../develop/reference-breaking-changes.md#looping-clients-will-be-interrupted).

---

### <a name="new-audit-logs-user-experience-now-available"></a>Ya disponible la nueva experiencia de usuario de los registros de auditoría

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Hemos creado una nueva página **Registros de auditoría** de Azure AD para mejorar la legibilidad y la búsqueda de información. Para ver la página **Registros de auditoría**, seleccione **Registros de auditoría** en la sección **Actividad** de Azure AD.

![Nueva página Registros de auditoría con información de ejemplo](media/whats-new/audit-logs-page.png)

Para obtener más información sobre la página **Registros de auditoría**, vea [Informes de actividad de auditoría en el portal de Azure Active Directory](../reports-monitoring/concept-audit-logs.md).

---

### <a name="new-warnings-and-guidance-to-help-prevent-accidental-administrator-lockout-from-misconfigured-conditional-access-policies"></a>Nuevas advertencias e instrucciones para ayudar a evitar el bloqueo accidental del administrador por una mala configuración de las directivas de acceso condicional

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

Para evitar que los administradores se bloqueen accidentalmente a sí mismos en sus propios inquilinos a causa de una mala configuración de las directivas de acceso, hemos creado nuevas advertencias y hemos actualizado las instrucciones en Azure Portal. Para obtener más información sobre las nuevas instrucciones, vea [¿Cuáles son las dependencias del servicio de acceso condicional de Azure Active Directory?](../conditional-access/service-dependencies.md).

---

### <a name="improved-end-user-terms-of-use-experiences-on-mobile-devices"></a>Experiencia mejorada de las condiciones de uso en dispositivos móviles

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Términos de uso Gobernanza

Hemos actualizado y mejorado el modo en el que los usuarios leen y aceptan las condiciones de uso en un dispositivo móvil. Ahora es posible ampliar y reducir el texto, volver atrás, descargar la información y seleccionar hipervínculos. Para obtener más información sobre las nuevas condiciones de uso, vea [Característica Condiciones de uso de Azure Active Directory](../conditional-access/terms-of-use.md#what-terms-of-use-looks-like-for-users).

---

### <a name="new-azure-ad-activity-logs-download-experience-available"></a>Disponible la nueva experiencia de descarga de los registros de actividad de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Ahora se pueden descargar grandes cantidades de registros de actividad directamente desde Azure Portal. Este cambio le permite:

- Descargar hasta 250 000 filas.

- Recibir una notificación cuando se complete la descarga.

- Personalizar el nombre del archivo.

- Determinar el formato de salida como JSON o CSV.

Para más información acerca de esta característica, consulte [Inicio rápido: Descarga de un informe de auditoría mediante Azure Portal](../reports-monitoring/quickstart-download-audit-report.md).

---

### <a name="breaking-change-updates-to-condition-evaluation-by-exchange-activesync-eas"></a>Cambio importante: modificaciones en la evaluación de estado de Exchange ActiveSync (EAS)

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Acceso condicional Control de acceso

Estamos actualizando el modo en que Exchange ActiveSync (EAS) evalúa las condiciones siguientes:

- Ubicación del usuario, basada en el país, la región o la dirección IP.

- Riesgo de inicio de sesión

- Plataforma de dispositivo

Si ha usado estas condiciones anteriormente en las directivas de acceso condicional, tenga en cuenta que el comportamiento de la condición puede cambiar. Por ejemplo, si ha usado la condición de ubicación de usuario en una directiva, es posible que la directiva se omita ahora en función de la ubicación del usuario.

---

## <a name="february-2019"></a>Febrero de 2019

### <a name="configurable-azure-ad-saml-token-encryption-public-preview"></a>Cifrado de tokens SAML de Azure AD configurable (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Ahora se puede configurar cualquier aplicación SAML compatible para que reciba tokens SAML cifrados. Cuando se configura y se usa con una aplicación, Azure AD cifra las aserciones SAML emitidas mediante una clave pública que se obtiene de un certificado almacenado en Azure AD.

Para obtener más información sobre cómo configurar el cifrado de tokens SAML, vea [Configuración del cifrado de tokens SAML de Azure AD](../manage-apps/howto-saml-token-encryption.md).

---

### <a name="create-an-access-review-for-groups-or-apps-using-azure-ad-access-reviews"></a>Creación de una revisión de acceso para grupos o aplicaciones con las revisiones de acceso de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Revisiones de acceso Gobernanza

Ahora se pueden incluir varios grupos o aplicaciones en una única revisión de acceso de Azure AD para comprobar la pertenencia a grupos o la asignación de aplicaciones. Las revisiones de acceso con varios grupos o aplicaciones se configuran usando los mismos valores y todos los revisores incluidos son notificados al mismo tiempo.

Para obtener más información sobre cómo crear una revisión de acceso con las revisiones acceso de Azure AD, vea [Creación de una revisión de acceso de grupos o aplicaciones con las revisiones de acceso de Azure AD](../governance/create-access-review.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---february-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (febrero de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En febrero de 2019, hemos agregado estas 27 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[Euromonitor Passport](../saas-apps/euromonitor-passport-tutorial.md), [MindTickle](../saas-apps/mindtickle-tutorial.md), [FAT FINGER](https://seeforgetest-exxon.azurewebsites.net/Account/create?Length=7), [AirStack](../saas-apps/airstack-tutorial.md), [Oracle Fusion ERP](../saas-apps/oracle-fusion-erp-tutorial.md), [IDrive](../saas-apps/idrive-tutorial.md), [Skyward Qmlativ](../saas-apps/skyward-qmlativ-tutorial.md), [Brightidea](../saas-apps/brightidea-tutorial.md), [AlertOps](../saas-apps/alertops-tutorial.md), [Soloinsight-CloudGate SSO](../saas-apps/soloinsight-cloudgate-sso-tutorial.md), Permission Click, [Brandfolder](../saas-apps/brandfolder-tutorial.md), [StoregateSmartFile](../saas-apps/smartfile-tutorial.md), [Pexip](../saas-apps/pexip-tutorial.md), [Stormboard](../saas-apps/stormboard-tutorial.md), [Seismic](../saas-apps/seismic-tutorial.md), [Share A Dream](https://www.shareadream.org/how-it-works), [Bugsnag](../saas-apps/bugsnag-tutorial.md), [webMethods Integration Cloud](../saas-apps/webmethods-integration-cloud-tutorial.md), [Knowledge Anywhere LMS](../saas-apps/knowledge-anywhere-lms-tutorial.md), [OU Campus](../saas-apps/ou-campus-tutorial.md), [Periscope Data](../saas-apps/periscope-data-tutorial.md), [Netop Portal](../saas-apps/netop-portal-tutorial.md), [smartvid.io](../saas-apps/smartvid.io-tutorial.md), [PureCloud by Genesys](../saas-apps/purecloud-by-genesys-tutorial.md) y [ClickUp Productivity Platform](../saas-apps/clickup-productivity-platform-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="enhanced-combined-mfasspr-registration"></a>Mejora del registro MFA/SSPR combinado

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Autenticación de usuarios

En respuesta a los comentarios de los clientes, se ha mejorado la experiencia en versión preliminar del registro combinado de MFA/SSPR para que los usuarios puedan registrar de forma más rápida su información de seguridad para MFA y SSPR.

**Si quiere activar ahora la experiencia mejorada para los usuarios, siga estos pasos:**

1. Inicie sesión como administrador global o administrador de usuarios en Azure Portal y vaya a **Azure Active Directory > Configuración de usuario > Administrar la configuración de las características en versión preliminar del panel de acceso**.

2. En la opción **Usuarios que pueden utilizar las características en versión preliminar para registrar y administrar la información de seguridad – actualizar**, elija activar las características para un **Grupo seleccionado de usuarios** o para **Todos los usuarios**.

A lo largo de las próximas semanas se retirará la capacidad de activar la antigua experiencia combinada de vista previa de registro MFA/SSPR para los inquilinos que todavía no la hayan activado.

**Para ver si el control se retirará en su inquilino, siga estos pasos:**

1. Inicie sesión como administrador global o administrador de usuarios en Azure Portal y vaya a **Azure Active Directory > Configuración de usuario > Administrar la configuración de las características en versión preliminar del panel de acceso**.

2. Si la opción **Usuarios que pueden utilizar las características en versión preliminar para registrar y administrar la información de seguridad** está establecida en **Ninguno**, la opción se retirará del inquilino.

Independientemente de si con anterioridad se ha activado la antigua experiencia combinada de vista previa de registro MFA/SSPR, la antigua experiencia se desactivará en una fecha futura. Por ello, es muy recomendable que cambie a la nueva experiencia lo antes posible.

Para obtener más información sobre la experiencia de registro mejorada, vea la entrada de blog sobre las [interesantes mejoras en la experiencia de registro combinado MFA y restablecimiento de contraseña de Azure AD](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Cool-enhancements-to-the-Azure-AD-combined-MFA-and-password/ba-p/354271).

---

### <a name="updated-policy-management-experience-for-user-flows"></a>Experiencia actualizada de la administración de directivas para flujos de usuario

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Se ha actualizado el proceso de creación y administración de directivas para facilitar los flujos del usuario (anteriormente conocidos como directivas integradas). Esta experiencia nueva es ahora la predeterminada para todos los inquilinos de Azure AD.

Si quiere hacer algún otro comentario o sugerencia, use los iconos de sonrisa o desaprobación del área **Envíenos comentarios** en la parte superior de la pantalla del portal.

Para obtener más información sobre la nueva experiencia de administración de directivas, vea la entrada de blog [Azure AD B2C tiene ahora la personalización de JavaScript y muchas más características nuevas](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595).

---

### <a name="choose-specific-page-element-versions-provided-by-azure-ad-b2c"></a>Elección de versiones específicas de elementos de página proporcionadas por Azure AD B2C

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Ahora puede elegir una versión específica de los elementos de página proporcionados por Azure AD B2C. Si selecciona una versión específica, puede probar las actualizaciones antes de que aparezcan en la página y predecir el comportamiento. Además, ahora puede decidir aplicar versiones específicas de la página para permitir las personalizaciones de JavaScript. Para activar esta característica, vaya a la página **Propiedades** en los flujos de usuario.

Para obtener más información sobre cómo elegir versiones específicas de los elementos de la página, vea la entrada de blog [Azure AD B2C tiene ahora la personalización de JavaScript y muchas más características nuevas](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595).

---

### <a name="configurable-end-user-password-requirements-for-b2c-ga"></a>Requisitos de contraseña del usuario final configurables para B2C (GA)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Ya es posible configurar la complejidad de las contraseñas que usan los usuarios finales de la organización, en lugar de tener que utilizar la directiva nativa de contraseñas de Azure AD. En la hoja **Propiedades** de los flujos de usuario (antes conocidos como directivas integradas), puede elegir una complejidad de contraseña **Simple** o **Segura**, o bien puede crear un conjunto de requisitos **Personalizado**.

Para obtener más información sobre la configuración de los requisitos de complejidad de contraseñas, vea [Configuración de los requisitos de complejidad de contraseñas de Azure Active Directory B2C](../../active-directory-b2c/password-complexity.md).

---

### <a name="new-default-templates-for-custom-branded-authentication-experiences"></a>Nuevas plantillas predeterminadas para experiencias de autenticación con marca personalizada

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Puede usar nuestras nuevas plantillas predeterminadas, ubicadas en la hoja **Diseños de página** de los flujos de usuario (anteriormente conocidos como directivas integradas), para que los usuarios disfruten de una experiencia de autenticación adaptada a su marca.

Para obtener más información sobre el uso de las plantillas, vea [Azure AD B2C tiene ahora la personalización de JavaScript y muchas más características nuevas](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595).

---

## <a name="january-2019"></a>Enero de 2019

### <a name="active-directory-b2b-collaboration-using-one-time-passcode-authentication-public-preview"></a>Colaboración B2B de Active Directory mediante la autenticación con código de acceso de un solo uso (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C

Se ha introducido la autenticación con código de acceso de un solo uso (OTP) para los usuarios invitados de B2B que no pueden autenticarse por otros medios, como Azure AD, una cuenta Microsoft (MSA) o la federación de Google. Este nuevo método de autenticación significa que los usuarios invitados no tienen que crear una nueva cuenta Microsoft. En cambio, al canjear una invitación o acceder a un recurso compartido, un usuario invitado puede solicitar el envío de un código temporal a una dirección de correo electrónico. Con este código temporal, el usuario invitado puede seguir iniciando sesión.

Para más información, consulte [Autenticación con código de acceso de un solo uso de correo electrónico (versión preliminar)](../external-identities/one-time-passcode.md) y el blog [Azure AD makes sharing and collaboration seamless for any user with any account](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-makes-sharing-and-collaboration-seamless-for-any-user/ba-p/325949) (Azure AD simplifica el uso compartido y la colaboración para cualquier usuario con una cuenta).

### <a name="new-azure-ad-application-proxy-cookie-settings"></a>Nueva configuración de cookies de Azure AD Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Hemos incluido tres nuevas configuraciones de cookies, disponibles para las aplicaciones que se publican a través de Application Proxy:

- **Usar cookie solo HTTP.** Establece la marca **HTTPOnly** en las cookies de sesión y acceso de Application Proxy. La activación de esta configuración ofrece ventajas de seguridad adicionales, como ayuda para evitar la copia o modificación de cookies por medio de scripting del lado cliente. Se recomienda activar esta marca (elija **Sí**) para disfrutar de estas ventajas.

- **Usar cookies seguras.** Establece la marca **Segura** en las cookies de sesión y acceso de Application Proxy. La activación de esta configuración ofrece ventajas de seguridad adicionales, como la garantía de que las cookies se transmiten solo a través de canales seguros de TLS, como HTTPS. Se recomienda activar esta marca (elija **Sí**) para disfrutar de estas ventajas.

- **Usar cookies persistentes.** Impide que las cookies de acceso expiren cuando se cierra el explorador web. Estas cookies se mantienen vigentes durante toda la duración del token de acceso. Sin embargo, las cookies se restablecen si se alcanza la hora de expiración o si el usuario elimina manualmente la cookie. Se recomienda que mantenga la configuración predeterminada **No**, activando el ajuste solo para las aplicaciones anteriores que no comparten cookies entre procesos.

Para obtener más información acerca de las nuevas cookies, consulte [Configuración de las cookies para el acceso a aplicaciones locales en Azure Active Directory](../manage-apps/application-proxy-configure-cookie-settings.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---january-2019"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD (enero de 2019)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En enero de 2019, hemos agregado estas 35 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[Firstbird](../saas-apps/firstbird-tutorial.md), [Folloze](../saas-apps/folloze-tutorial.md), [Talent Palette](../saas-apps/talent-palette-tutorial.md), [Infor CloudSuite](../saas-apps/infor-cloud-suite-tutorial.md), [Cisco Umbrella](../saas-apps/cisco-umbrella-tutorial.md), [Zscaler Internet Access Administrator](../saas-apps/zscaler-internet-access-administrator-tutorial.md), [Expiration Reminder](../saas-apps/expiration-reminder-tutorial.md), [InstaVR Viewer](../saas-apps/instavr-viewer-tutorial.md), [CorpTax](../saas-apps/corptax-tutorial.md), [Verb](https://app.verb.net/login), [OpenLattice](https://openlattice.com/agora), [TheOrgWiki](https://www.theorgwiki.com/signup), [Pavaso Digital Close](../saas-apps/pavaso-digital-close-tutorial.md), [GoodPractice Toolkit](../saas-apps/goodpractice-toolkit-tutorial.md), [Cloud Service PICCO](../saas-apps/cloud-service-picco-tutorial.md), [AuditBoard](../saas-apps/auditboard-tutorial.md), [iProva](../saas-apps/iprova-tutorial.md), [Workable](../saas-apps/workable-tutorial.md), [CallPlease](https://webapp.callplease.com/create-account/create-account.html), [GTNexus SSO System](../saas-apps/gtnexus-sso-module-tutorial.md), [CBRE ServiceInsight](../saas-apps/cbre-serviceinsight-tutorial.md), [Deskradar](../saas-apps/deskradar-tutorial.md), [Coralogixv](../saas-apps/coralogix-tutorial.md), [Signagelive](../saas-apps/signagelive-tutorial.md), [ARES for Enterprise](../saas-apps/ares-for-enterprise-tutorial.md), [K2 for Office 365](https://www.k2.com/O365), [Xledger](https://www.xledger.net/), [iDiD Manager](../saas-apps/idid-manager-tutorial.md), [HighGear](../saas-apps/highgear-tutorial.md), [Visitly](../saas-apps/visitly-tutorial.md), [Korn Ferry ALP](../saas-apps/korn-ferry-alp-tutorial.md), [Acadia](../saas-apps/acadia-tutorial.md) y [Adoddle cSaas Platform](../saas-apps/adoddle-csaas-platform-tutorial.md)<!-- , [CaféX Portal (Meetings)](https://docs.microsoft.com/azure/active-directory/saas-apps/cafexportal-meetings-tutorial), [MazeMap Link](https://docs.microsoft.com/azure/active-directory/saas-apps/mazemaplink-tutorial)-->

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-azure-ad-identity-protection-enhancements-public-preview"></a>Nuevas mejoras de Azure AD Identity Protection (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

Nos complace anunciar que hemos agregado las siguientes mejoras a la oferta de versión preliminar pública de Azure AD Identity Protection:

- Una interfaz de usuario actualizada y más integrada

- API adicionales

- Evaluación de riesgos mejorada a través de aprendizaje automático

- Alineación de todo el producto entre usuarios e inicios de sesión no seguros

Para obtener más información sobre las mejoras, consulte [What is Azure Active Directory Identity Protection (refreshed)?](../identity-protection/overview-identity-protection.md) (¿Qué es Azure Active Directory Identity Protection [actualizado]?) a fin de conocer los detalles y compartir sus ideas a través de los mensajes en el producto.

---

### <a name="new-app-lock-feature-for-the-microsoft-authenticator-app-on-ios-and-android-devices"></a>Nueva característica Bloqueo de aplicación para la aplicación Microsoft Authenticator en dispositivos iOS y Android

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicación Microsoft Authenticator Seguridad y protección de la identidad

Para proteger sus códigos de acceso de un solo uso, la información de aplicación y la configuración de la aplicación, puede activar la característica Bloqueo de aplicación en la aplicación Microsoft Authenticator. Al activar el Bloqueo de aplicación se le pedirá que se autentique con su PIN o características biométricas cada vez que abra la aplicación Microsoft Authenticator.

Para obtener más información, vea las [Preguntas más frecuentes de la aplicación Microsoft Authenticator](../user-help/user-help-auth-app-faq.md).

---

### <a name="enhanced-azure-ad-privileged-identity-management-pim-export-capabilities"></a>Mejora de las capacidades de exportación de Azure AD Privileged Identity Management (PIM)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Los administradores de Privileged Identity Management (PIM) ahora pueden exportar todas las asignaciones de roles activas y válidas para un recurso específico, lo cual incluye las asignaciones de roles para todos los recursos secundarios. Anteriormente, a los administradores les resultaba difícil obtener una lista completa de asignaciones de roles para una suscripción y tenían que exportar dichas asignaciones para cada recurso específico.

Para obtener más información, consulte [Visualización de la actividad y del historial de auditoría para los roles de recursos de Azure en PIM](../privileged-identity-management/azure-pim-resource-rbac.md).

---

## <a name="novemberdecember-2018"></a>Noviembre/diciembre de 2018

### <a name="users-removed-from-synchronization-scope-no-longer-switch-to-cloud-only-accounts"></a>Los usuarios que se quitaron del ámbito de sincronización ya no podrán cambiarse a cuentas solo en la nube

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Administración de usuarios Directorio

>[!Important]
>Hemos escuchado y comprendido la frustración que les genera esta corrección. Por lo tanto, hemos revertido este cambio hasta el momento en que podamos ofrecerle una implementación más sencilla para su organización.

Se ha corregido un error en el que la marca DirSyncEnabled de un usuario se cambiaba erróneamente a **False** cuando el objeto de Active Directory Domain Services se excluía del ámbito de sincronización y, después, se movía a la Papelera de reciclaje de Azure AD en el siguiente ciclo de sincronización. Como resultado de esta corrección, si el usuario se excluye del ámbito de sincronización y posteriormente se restaura desde la Papelera de reciclaje de Azure AD, la cuenta de dicho usuario permanece sincronizada desde la instancia local de AD, según lo previsto, y no se puede administrar en la nube, ya que su origen de autoridad (SoA) permanece como AD local.

Antes de esta corrección, había un problema cuando se cambiaba la marca DirSyncEnabled a False. Daba la impresión errónea de que estas cuentas se convertían en objetos solo en la nube y de que las cuentas se podían administrar en la nube. Sin embargo, las cuentas siguen conservando su SoA como propiedades locales y sincronizadas (atributos paralelos) procedentes de la instancia local de AD. Esta situación provocó varios problemas en Azure AD y en otras cargas de trabajo en la nube (como Exchange Online), ya que esperaban tratar estas cuentas como sincronizadas desde AD y, sin embargo, se comportaban como cuentas solo en la nube.

En este momento, la única manera de convertir realmente una cuenta sincronizada de AD en una cuenta de solo nube, es deshabilitando DirSync en el nivel del inquilino, lo que desencadena una operación de back-end para transferir el SoA. Este tipo de cambio de SoA requiere (pero no se limita a) la limpieza de todos los atributos locales relacionados (como los atributos paralelos y de LastDirSyncTime) y el envío de una señal a otras cargas de trabajo en la nube para convertir también su objeto respectivo en una cuenta solo en la nube.

Por lo tanto, esta corrección evita las actualizaciones directas sobre el atributo ImmutableID de un usuario sincronizado desde AD, lo que era necesario en algunos escenarios en el pasado. Por diseño, el atributo ImmutableID de un objeto en Azure AD, como su nombre indica, es inmutable. Existen nuevas características implementadas en los clientes de sincronización de Azure AD Connect y Azure AD Connect Health para abordar estos escenarios:

- **Actualizaciones del atributo ImmutableID a gran escala para muchos usuarios por etapas**

  Por ejemplo, debe realizar una migración entre bosques de AD DS larga. Solución: Use Azure AD Connect para la **Configuración del delimitador de origen** y, mientras el usuario migra, copie los valores del atributo ImmutableID existentes de Azure AD al atributo DS-Consistency-Guid del usuario de AD DS local del nuevo bosque. Para obtener más información, consulte [Uso de msDS-ConsistencyGuid como sourceAnchor](../hybrid/plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor).

- **Actualizaciones del atributo ImmutableID a gran escala para muchos usuarios de una sola vez**

  Por ejemplo, al implementar Azure AD Connect se cometió un error, y ahora es necesario cambiar el atributo SourceAnchor. Solución: Deshabilite DirSync en el nivel del inquilino y borre todos los valores de ImmutableID no válidos. Para obtener más información, vea [Desactivar la sincronización de directorios para Office 365](/office365/enterprise/turn-off-directory-synchronization).

- **Hacer coincidir un usuario local con un usuario existente de Azure AD** Por ejemplo, un usuario que se haya vuelto a crear en AD DS genera un duplicado en la cuenta de Azure AD en lugar de reasignarlo a una cuenta de Azure AD existente (objeto huérfano). Solución: Use Azure AD Connect Health en Azure Portal para reasignar el delimitador de origen o el atributo ImmutableID. Para obtener más información, consulte [Escenario del objeto huérfano](../hybrid/how-to-connect-health-diagnose-sync-errors.md#orphaned-object-scenario).

### <a name="breaking-change-updates-to-the-audit-and-sign-in-logs-schema-through-azure-monitor"></a>Cambio de última hora: Actualizaciones de los esquemas de registros de auditoría y de inicio de sesión mediante Azure Monitor

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Informes Supervisión e informes

Actualmente, estamos publicando los flujos de registro de auditorías e inicios de sesión mediante Azure Monitor, por lo que puede integrar fácilmente los archivos de registro con sus herramientas SIEM o con Log Analytics. Basándonos en sus comentarios, y en preparación para el anuncio de disponibilidad general para esta característica, vamos a realizar los siguientes cambios en el esquema. Estos cambios en los esquemas y las actualizaciones de documentación relacionadas se producirán en la primera semana de enero.

#### <a name="new-fields-in-the-audit-schema"></a>Nuevos campos en el esquema de auditoría
Vamos a agregar un nuevo campo **Tipo de operación**, para proporcionar el tipo de operación que se realiza en el recurso. Por ejemplo, **Agregar**, **Actualizar** o **Eliminar**.

#### <a name="changed-fields-in-the-audit-schema"></a>Campos modificados en el esquema de auditoría
Los siguientes campos han cambiado en el esquema de auditoría:

|Nombre del campo|Qué cambia|Valores anteriores|Nuevos valores|
|----------|------------|----------|----------|
|Category|Este era el campo **Nombre de servicio**. Ahora es el campo **Categorías de auditoría**. A **Nombre de servicio** se le ha cambiado el nombre por el de campo **loggedByService**.|<ul><li>Account Provisioning (Aprovisionamiento de cuentas)</li><li>Core Directory (Directorio principal)</li><li>Autoservicio de restablecimiento de contraseña</li></ul>|<ul><li>User Management</li><li>Administración de grupos</li><li>Administración de la aplicación</li></ul>|
|targetResources|Incluye **TargetResourceType** en el nivel superior.|&nbsp;|<ul><li>Directiva</li><li>Aplicación</li><li>Usuario</li><li>Grupo</li></ul>|
|loggedByService|Proporciona el nombre del servicio que generó el registro de auditoría.|Null|<ul><li>Account Provisioning (Aprovisionamiento de cuentas)</li><li>Core Directory (Directorio principal)</li><li>Restablecimiento de la contraseña de autoservicio</li></ul>|
|Resultado|Proporciona el resultado de los registros de auditoría. Anteriormente, este aparecía en una lista, pero ahora se muestra el valor real.|<ul><li>0</li><li>1</li></ul>|<ul><li>Correcto</li><li>Error</li></ul>|

#### <a name="changed-fields-in-the-sign-in-schema"></a>Campos modificados en el esquema de inicio de sesión
Los siguientes campos han cambiado en el esquema de inicio de sesión:

|Nombre del campo|Qué cambia|Valores anteriores|Nuevos valores|
|----------|------------|----------|----------|
|appliedConditionalAccessPolicies|Este era el campo **conditionalaccessPolicies**. Ahora es el campo **appliedConditionalAccessPolicies**.|Sin cambios|Sin cambios|
|conditionalAccessStatus|Proporciona el resultado del estado de la directiva de acceso condicional en el inicio de sesión. Anteriormente, este aparecía en una lista, pero ahora se muestra el valor real.|<ul><li>0</li><li>1</li><li>2</li><li>3</li></ul>|<ul><li>Correcto</li><li>Error</li><li>No aplicado</li><li>Disabled</li></ul>|
|appliedConditionalAccessPolicies: result|Proporciona el resultado del estado individual de la directiva de acceso condicional en el inicio de sesión. Anteriormente, este aparecía en una lista, pero ahora se muestra el valor real.|<ul><li>0</li><li>1</li><li>2</li><li>3</li></ul>|<ul><li>Correcto</li><li>Error</li><li>No aplicado</li><li>Disabled</li></ul>|

Para más información acerca del esquema, consulte [Interpretación del esquema de registros de auditoría de Azure AD en Azure Monitor (versión preliminar)](../reports-monitoring/reference-azure-monitor-audit-log-schema.md).

---

### <a name="identity-protection-improvements-to-the-supervised-machine-learning-model-and-the-risk-score-engine"></a>Mejoras de la protección de identidades en el modelo de Machine Learning supervisado y en el motor de puntuaciones de riesgo

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Identity Protection Puntuaciones de riesgo

Las mejoras en el usuario relacionadas con la protección de identidades y en el motor de evaluación de riesgos de inicio de sesión pueden ayudar a mejorar la precisión y cobertura de los riesgos del usuario. Puede que los administradores hayan observado que el nivel de riesgo del usuario ya no está vinculado directamente al nivel de riesgo de detecciones específicas y que hay un aumento en el número y en el nivel de los eventos de inicio de sesión de riesgo.

El modelo de Machine Learning supervisado es el encargado ahora de evaluar las detecciones de riesgo. Este calcula el riesgo del usuario usando características adicionales de los inicios de sesión del usuario y un patrón de detecciones. Según este modelo, el administrador puede detectar usuarios con puntuaciones de riesgo altas, incluso aunque las detecciones asociadas con ese usuario sean de nivel bajo o medio.

---

### <a name="administrators-can-reset-their-own-password-using-the-microsoft-authenticator-app-public-preview"></a>Los administradores pueden restablecer sus propias contraseñas mediante la aplicación Microsoft Authenticator (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Autenticación de usuarios

Los administradores de Azure AD ya pueden restablecer sus propias contraseñas mediante las notificaciones de la aplicación Microsoft Authenticator o mediante un código de cualquier aplicación de autenticación móvil, o token de hardware. Para restablecer sus propias contraseñas, los administradores ahora pueden usar dos de los métodos siguientes:

- Notificación de la aplicación Microsoft Authenticator

- Un código de otra aplicación de autenticación móvil o un token de hardware

- Email

- llamada de teléfono

- mensaje de texto

Para más información sobre el uso de la aplicación Microsoft Authenticator para restablecer contraseñas, consulte [Autoservicio de restablecimiento de contraseña de Azure AD: Aplicación móvil y SSPR (versión preliminar)](../authentication/concept-sspr-howitworks.md#mobile-app-and-sspr)

---

### <a name="new-azure-ad-cloud-device-administrator-role-public-preview"></a>Nuevo rol de administrador de dispositivos en la nube de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración y registro de dispositivos Control de acceso

Los administradores pueden asignar usuarios al nuevo rol de administrador de dispositivos en la nube para realizar las tareas propias de este rol. Los usuarios asignados a este rol pueden habilitar, deshabilitar y eliminar dispositivos en Azure AD y leer las claves de BitLocker de Windows 10 (si las hay) en Azure Portal.

Para más información acerca de los roles y permisos, consulte [Asignación de roles de administrador en Azure Active Directory](../roles/permissions-reference.md).

---

### <a name="manage-your-devices-using-the-new-activity-timestamp-in-azure-ad-public-preview"></a>Administración de dispositivos con la nueva marca de tiempo de actividad de Azure AD (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración y registro de dispositivos Administración del ciclo de vida de dispositivos

Somos conscientes de que, con el paso del tiempo, debe actualizar y retirar los dispositivos de su organización en Azure AD para evitar tener dispositivos obsoletos en su entorno. Para ayudarle con este proceso, Azure AD ya actualiza los dispositivos con una nueva marca de tiempo de actividad que le ayuda a administrar el ciclo de vida del dispositivo.

Para más información acerca de cómo obtener y usar esta marca de tiempo, consulte [Procedimiento: Administración de dispositivos obsoletos en Azure AD](../devices/manage-stale-devices.md).

---

### <a name="administrators-can-require-users-to-accept-a-terms-of-use-on-each-device"></a>Los administradores pueden requerir que los usuarios acepten los términos de uso en cada dispositivo

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Gobernanza

Los administradores ahora pueden activar la opción **Requerir que los usuarios concedan su consentimiento en todos los dispositivos** para requerir que los usuarios acepten los términos de uso en todos los dispositivos que estén usando en su inquilino.

Para obtener más información, consulte [la sección de términos de uso por dispositivo del artículo Característica Términos de uso de Azure Active Directory](../conditional-access/terms-of-use.md#per-device-terms-of-use).

---

### <a name="administrators-can-configure-a-terms-of-use-to-expire-based-on-a-recurring-schedule"></a>Los administradores pueden configurar la expiración de los términos de uso mediante una programación periódica

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Gobernanza


Los administradores pueden ahora activar la opción **Expirar autorizaciones** para hacer que los términos de uso expiren para todos los usuarios según la programación periódica especificada. La programación puede ser anual, semestral, trimestral o mensual. Una vez que los términos de uso expiran, los usuarios deben volver a aceptarlos.

Para obtener más información, consulte [la sección Agregar términos de uso del artículo Característica Términos de uso de Azure Active Directory](../conditional-access/terms-of-use.md#add-terms-of-use).

---

### <a name="administrators-can-configure-a-terms-of-use-to-expire-based-on-each-users-schedule"></a>Los administradores pueden configurar la expiración de los términos de uso según la programación de cada usuario

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Gobernanza

Los administradores ahora pueden especificar una duración tras la cual el usuario debe volver a aceptar los términos de uso. Por ejemplo, los administradores pueden especificar que los usuarios deben volver a aceptar los términos de uso cada 90 días.

Para obtener más información, consulte [la sección Agregar términos de uso del artículo Característica Términos de uso de Azure Active Directory](../conditional-access/terms-of-use.md#add-terms-of-use).

---

### <a name="new-azure-ad-privileged-identity-management-pim-emails-for-azure-active-directory-roles"></a>Nuevos correos electrónicos de Azure Active Directory Privileged Identity Management (PIM) para los roles de Azure Active Directory

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Los clientes que usan Azure AD Privileged Identity Management (PIM) ahora pueden recibir un correo electrónico de resumen semanal que incluye la información siguiente de los últimos siete días:

- Introducción a las asignaciones de roles principales y permanentes

- Número de usuarios que activan roles

- Número de usuarios asignados a roles en PIM

- Número de usuarios asignados a roles fuera de PIM

- Número de usuarios a los que se ha "convertido en permanentes" en PIM

Para más información sobre PIM y las notificaciones de correo electrónico disponibles, consulte [Notificaciones por correo electrónico en PIM](../privileged-identity-management/pim-email-notifications.md).

---

### <a name="group-based-licensing-is-now-generally-available"></a>Las licencias basadas en grupos tienen ahora disponibilidad general

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Directorio

Las licencias basadas en grupos ya no están en la fase de versión preliminar pública sino en la de disponibilidad general. Como parte de esta versión general, se ha logrado que esta característica sea más escalable y se ha agregado la posibilidad de volver a procesar las asignaciones de licencias basadas en grupos para un solo usuario y la posibilidad de usar licencias basadas en grupos con licencias E3/A3 de Office 365.

Para más información acerca de las licencias basadas en grupos, consulte [¿En qué consisten las licencias basadas en grupos de Azure Active Directory?](./active-directory-licensing-whatis-azure-portal.md)

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---november-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: noviembre de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En noviembre de 2018, hemos agregado 26 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[CoreStack](https://cloud.corestack.io/site/login), [HubSpot](../saas-apps/hubspot-tutorial.md), [GetThere](../saas-apps/getthere-tutorial.md), [Gra-Pe](../saas-apps/grape-tutorial.md), [eHour](https://getehour.com/try-now), [Consent2Go](../saas-apps/consent2go-tutorial.md), [Appinux](../saas-apps/appinux-tutorial.md), [DriveDollar](https://azuremarketplace.microsoft.com/marketplace/apps/savitas.drivedollar-azuread?tab=Overview), [Useall](../saas-apps/useall-tutorial.md), [Infinite Campus](../saas-apps/infinitecampus-tutorial.md), [Alaya](https://alayagood.com), [HeyBuddy](../saas-apps/heybuddy-tutorial.md), [Wrike SAML](../saas-apps/wrike-tutorial.md), [Drift](../saas-apps/drift-tutorial.md), [Zenegy for Business Central 365](https://accounting.zenegy.com/), [Everbridge Member Portal](../saas-apps/everbridge-tutorial.md), [IDEO](https://profile.ideo.com/users/sign_up), [Ivanti Service Manager (ISM)](../saas-apps/ivanti-service-manager-tutorial.md), [Peakon](../saas-apps/peakon-tutorial.md), [Allbound SSO](../saas-apps/allbound-sso-tutorial.md), [Plex Apps - Classic Test](https://test.plexonline.com/signon), [Plex Apps – Classic](https://www.plexonline.com/signon), [Plex Apps - UX Test](https://test.cloud.plex.com/sso), [Plex Apps – UX](https://cloud.plex.com/sso), [Plex Apps – IAM](https://accounts.plex.com/), [CRAFTS - Childcare Records, Attendance, & Financial Tracking System](https://getcrafts.ca/craftsregistration)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

## <a name="october-2018"></a>Octubre de 2018

### <a name="azure-ad-logs-now-work-with-azure-log-analytics-public-preview"></a>Los registros de Azure AD ahora funcionan con Azure Log Analytics (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Nos complace anunciar que ahora puede reenviar los registros de Azure AD a Azure Log Analytics. Esta característica tan solicitada le ayuda a obtener un acceso aún mejor al análisis para su negocio, las operaciones y la seguridad, además de contribuir a supervisar la infraestructura. Para más información, consulte el blog sobre [los registros de actividad de Azure Active Directory en Azure Log Analytics que ya está disponible](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-Active-Directory-Activity-logs-in-Azure-Log-Analytics-now/ba-p/274843).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---october-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: octubre de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En octubre de 2018, hemos agregado 14 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[My Award Points](../saas-apps/myawardpoints-tutorial.md), [Vibe HCM](../saas-apps/vibehcm-tutorial.md), ambyint, [MyWorkDrive](../saas-apps/myworkdrive-tutorial.md), [BorrowBox](../saas-apps/borrowbox-tutorial.md), Dialpad, [ON24 Virtual Environment](../saas-apps/on24-tutorial.md), [RingCentral](../saas-apps/ringcentral-tutorial.md), [Zscaler Three](../saas-apps/zscaler-three-tutorial.md), [Phraseanet](../saas-apps/phraseanet-tutorial.md), [Appraisd](../saas-apps/appraisd-tutorial.md), [Workspot Control](../saas-apps/workspotcontrol-tutorial.md), [Shuccho Navi](../saas-apps/shucchonavi-tutorial.md), [Glassfrog](../saas-apps/glassfrog-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="azure-ad-domain-services-email-notifications"></a>Notificaciones por correo electrónico de Azure AD Domain Services

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Domain Services Azure AD Domain Services

Azure AD Domain Services proporciona alertas en Azure Portal sobre configuraciones incorrectas o problemas con el dominio administrado. Estas alertas incluyen guías paso a paso para que pueda intentar corregir los problemas sin tener que ponerse en contacto con el servicio de soporte técnico.

A partir de octubre, podrá personalizar la configuración de notificaciones para el dominio administrado por lo que, cuando se produzcan nuevas alertas, se enviará un correo electrónico a un grupo de personas designado, sin necesidad de tener que comprobar constantemente si en el portal hay actualizaciones.

Para más información, consulte [Configuración de notificaciones de Azure AD Domain Services](../../active-directory-domain-services/notifications.md).

---

### <a name="azure-ad-portal-supports-using-the-forcedelete-domain-api-to-delete-custom-domains"></a>El portal de Azure AD admite el uso de la API de dominio ForceDelete para eliminar los dominios personalizados

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Administración de directorios Directorio

Nos complace anunciar que ahora puede usar la API de dominio ForceDelete para eliminar los nombres de dominio personalizados cambiando las referencias de forma asincrónica, como los usuarios, los grupos y las aplicaciones del nombre de dominio personalizado (contoso.com) al nombre de dominio predeterminado inicial (contoso.onmicrosoft.com).

Este cambio ayuda a eliminar más rápidamente los nombres de dominio personalizados si la organización ya no utiliza el nombre o si necesita utilizar ese nombre de dominio con otra instancia de Azure AD.

Para más información, consulte [Eliminación de un nombre de dominio personalizado](../enterprise-users/domains-manage.md#delete-a-custom-domain-name).

---

## <a name="september-2018"></a>Septiembre de 2018

### <a name="updated-administrator-role-permissions-for-dynamic-groups"></a>Permisos de rol de administrador actualizados para grupos dinámicos

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Administración de grupos Colaboración

Se ha corregido un problema, por lo que determinados roles de administrador ya pueden crear y actualizar reglas de pertenencia dinámicas, sin necesidad de ser el propietario del grupo.

Los roles son:

- Administrador global

- Administrador de Intune

- Administrador de usuarios

Para más información, consulte [Creación de un grupo dinámico y comprobación de su estado](../enterprise-users/groups-create-rule.md)

---

### <a name="simplified-single-sign-on-sso-configuration-settings-for-some-third-party-apps"></a>Valores de configuración simplificada de inicio de sesión único (SSO) para algunas aplicaciones de terceros

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Somos conscientes de que la configuración del inicio de sesión único (SSO) para aplicaciones de SaaS (software como servicio) puede ser un desafío debido a la naturaleza única de la configuración de cada aplicación. Hemos creado una configuración simplificada para rellenar de forma automática los valores de configuración de SSO para las siguientes aplicaciones de SaaS de terceros:

- Zendesk

- ArcGis Online

- Jamf Pro

Para empezar a usar esta experiencia de un solo clic, vaya a la página **Azure Portal** > **Configuración de SSO** de la aplicación. Para más información, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md)

---

### <a name="azure-active-directory---where-is-your-data-located-page"></a>Página Azure Active Directory - Where is your data located? (Azure Active Directory: dónde se encuentran los datos)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Otros GoLocal

Seleccione la región de su empresa en la página **Azure Active Directory - Where is your data located?** (Azure Active Directory: dónde se encuentran los datos) para ver qué centro de datos de Azure alberga los datos en reposo de Azure AD para todos los servicios de Azure AD. Puede filtrar la información por los servicios específicos de Azure AD para la región de su empresa.

Para acceder a esta característica y para más información, consulte [Azure Active Directory - Where is your data located?](https://aka.ms/AADDataMap) (Azure Active Directory: dónde se encuentran los datos).

---

### <a name="new-deployment-plan-available-for-the-my-apps-access-panel"></a>Nuevo plan de implementación disponible para el panel de acceso a Mis aplicaciones

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones SSO

Consulte el nuevo plan de implementación disponible para el panel de acceso a Mis aplicaciones (https://aka.ms/deploymentplans).
El panel de acceso a Mis aplicaciones proporciona a los usuarios un único lugar en el que pueden buscar y acceder a sus aplicaciones. Este portal también proporciona a los usuarios oportunidades de autoservicio, como por ejemplo solicitar acceso a aplicaciones y grupos, o administrar el acceso a estos recursos en nombre de otros.

Para más información, consulte [¿Qué es el portal Mis aplicaciones?](../user-help/my-apps-portal-end-user-access.md)

---

### <a name="new-troubleshooting-and-support-tab-on-the-sign-ins-logs-page-of-the-azure-portal"></a>Pestaña nueva solución de problemas y soporte técnico de la página de registros de inicios de sesión de Azure Portal

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

El fin de la nueva pestaña **Solución de problemas y soporte técnico** de la página **Inicios de sesión** de Azure Portal es ayudar a los administradores e ingenieros de soporte técnico a solucionar problemas relacionados con los inicios de sesión de Azure AD. Esta nueva pestaña proporciona el código de error, mensaje de error y las recomendaciones de corrección (si existen) para ayudar a solucionar el problema. Si no puede resolver el problema, también le ofrecemos una nueva forma de crear una incidencia de soporte técnico mediante **Copiar al Portapapeles**, que rellena los campos **Id. de solicitud** y **Fecha (UTC)** del archivo de registro en su incidencia de soporte técnico.

![Registros de inicio de sesión que se muestran en la nueva pestaña](media/whats-new/troubleshooting-and-support.png)

---

### <a name="enhanced-support-for-custom-extension-properties-used-to-create-dynamic-membership-rules"></a>Compatibilidad mejorada con las propiedades de extensión personalizadas utilizadas para crear reglas de pertenencia dinámica

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Administración de grupos Colaboración

Con esta actualización, ahora puede hacer clic en el vínculo **Get custom extension properties** (Obtener propiedades de extensión personalizadas) desde el generador de reglas de grupo de usuarios dinámico, escriba el identificador de aplicación único y recibir la lista completa de las propiedades de extensión personalizadas para usarla al crear una regla de pertenencia dinámica para los usuarios. Esta lista también se puede actualizar esta lista para obtener nuevas propiedades de extensión personalizada para la aplicación.

Para más información acerca del uso de propiedades de extensión personalizadas para las reglas de pertenencia dinámica, consulte [Propiedades de extensión y propiedades de extensión personalizadas](../enterprise-users/groups-dynamic-membership.md#extension-properties-and-custom-extension-properties)

---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>Nuevas aplicaciones cliente aprobadas para el acceso condicional basado en aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

Las siguientes aplicaciones se incluyen en la lista de [aplicaciones cliente aprobadas](../conditional-access/concept-conditional-access-conditions.md#client-apps):

- Microsoft To-Do

- Microsoft Stream

Para más información, consulte:

- [Acceso condicional basado en aplicaciones de Azure AD](../conditional-access/app-based-conditional-access.md)

---

### <a name="new-support-for-self-service-password-reset-from-the-windows-7881-lock-screen"></a>Nueva compatibilidad con Autoservicio de restablecimiento de contraseña desde la pantalla de bloqueo de Windows 7/8/8.1

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** SSPR Autenticación de usuarios

Después de configurar esta nueva característica los usuarios verán un vínculo para restablecer su contraseña desde la pantalla de **bloqueo** de un dispositivo que ejecuta Windows 7, Windows 8 o Windows 8.1. Al hacer clic en ese vínculo, se guía al usuario por el mismo flujo de restablecimiento de contraseña que por el explorador web.

Para más información, consulte [How to: Enable password reset from Windows 7, 8, and 8.1](../authentication/howto-sspr-windows.md) (Procedimiento para habilitar el restablecimiento de contraseña desde Windows 7, 8, and 8.1)

---

### <a name="change-notice-authorization-codes-will-no-longer-be-available-for-reuse"></a>Aviso de cambio: Los códigos de autorización ya no estarán disponibles para su reutilización

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

A partir del 15 de noviembre de 2018, Azure AD dejará de aceptar los códigos de autenticación que se usaban anteriormente para las aplicaciones. Este cambio de seguridad ayuda a poner Azure AD en consonancia con la especificación de OAuth y se aplicará en los puntos de conexión v1 y v2.

Si la aplicación reutiliza códigos de autorización para obtener tokens para varios recursos, es recomendable que use el código para obtener un token de actualización y, a continuación, utilice este para adquirir tokens adicionales para otros recursos. Los códigos de autorización solo se pueden usar una vez, pero los tokens de actualización se pueden usar varias veces en varios recursos. Una aplicación que intente reutilizar un código de autenticación durante el flujo de código de OAuth obtendrá el error invalid_grant.

Para este y otros cambios relacionados con los protocolos, consulte [la lista completa de las novedades de la autenticación](../develop/reference-breaking-changes.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---september-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: septiembre de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En septiembre de 2018, hemos agregado 16 nuevas aplicaciones con compatibilidad con la federación a la galería de aplicaciones:

[Uberflip](../saas-apps/uberflip-tutorial.md), [Comeet Recruiting Software](../saas-apps/comeetrecruitingsoftware-tutorial.md), [Workteam](../saas-apps/workteam-tutorial.md), [ArcGIS Enterprise](../saas-apps/arcgisenterprise-tutorial.md), [Nuclino](../saas-apps/nuclino-tutorial.md), [JDA Cloud](../saas-apps/jdacloud-tutorial.md), [Snowflake](../saas-apps/snowflake-tutorial.md), NavigoCloud, [Figma](../saas-apps/figma-tutorial.md), join.me, [ZephyrSSO](../saas-apps/zephyrsso-tutorial.md), [Silverback](../saas-apps/silverback-tutorial.md), Riverbed Xirrus EasyPass, [Rackspace SSO](../saas-apps/rackspacesso-tutorial.md), Enlyft SSO for Azure, SurveyMonkey, [Convene](../saas-apps/convene-tutorial.md) y [dmarcian](../saas-apps/dmarcian-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="support-for-additional-claims-transformations-methods"></a>Compatibilidad con métodos de transformación de notificaciones adicionales

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Hemos introducido nuevos métodos de transformación de notificaciones, ToLower() y ToUpper(), que se pueden aplicar a los tokens SAML desde la página **Configuración del inicio de sesión único** basada en SAML.

Para más información, consulte [Procedimientos para personalizar las notificaciones emitidas en el token SAML para aplicaciones empresariales en Azure AD](../develop/active-directory-saml-claims-customization.md)

---

### <a name="updated-saml-based-app-configuration-ui-preview"></a>Interfaz de usuario de la configuración de aplicaciones basadas en SAML actualizada (versión preliminar)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Aplicaciones empresariales SSO

Como parte de nuestra interfaz de usuario de la configuración de aplicaciones basadas en SAML actualizada, obtendrá:

- Una experiencia actualizada del tutorial para configurar aplicaciones basadas en SAML.

- Mayor visibilidad de lo que falta o es incorrecto en la configuración.

- La capacidad para agregar varias direcciones de correo electrónico para la notificación de expiración de certificados.

- Nuevos métodos de transformación de notificaciones, ToLower() ToUpper(), y más.

- Una forma de cargar su propio certificado de firma de tokens para las aplicaciones empresariales.

- Una forma de establecer el formato de NameID para aplicaciones SAML y una forma de establecer el valor de NameID como extensiones de directorio.

Para activar esta vista actualizada, haga clic en el vínculo **Try out our new experience** (Probar nuestra nueva experiencia) en la parte superior de la página **Inicio de sesión único**. Para más información, consulte el [Tutorial: Configuración del inicio de sesión único basado en SAML para una aplicación con Azure Active Directory](../manage-apps/view-applications-portal.md).

---

## <a name="august-2018"></a>Agosto de 2018

### <a name="changes-to-azure-active-directory-ip-address-ranges"></a>Cambios realizados en los intervalos de direcciones IP de Azure Active Directory

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Otros Plataforma

Estamos introduciendo intervalos IP mayores en Azure AD, lo que significa que si ha configurado los intervalos IP de Azure AD para los firewalls, enrutadores o grupos de seguridad de red, deberá actualizarlos. Realizamos esta actualización para que no tenga que volver a cambiar su firewall, enrutador ni las configuraciones de intervalo IP de los grupos de seguridad de red cuando Azure AD agregue nuevos puntos de conexión.

El tráfico se moverá a estos nuevos intervalos durante los próximos dos meses. Para continuar con un servicio ininterrumpido, debe agregar estos valores actualizados a las direcciones IP antes del 10 de septiembre de 2018:

- 20.190.128.0/18

- 40.126.0.0/18

Se recomienda encarecidamente no quitar los intervalos IP antiguos hasta que todo el tráfico se haya movido a los nuevos intervalos. Para actualizaciones sobre el movimiento y saber cuándo quitar los intervalos antiguos, consulte [Office 365 URLs and IP address ranges](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2) (Direcciones URL e intervalos IP de Office 365).

---

### <a name="change-notice-authorization-codes-will-no-longer-be-available-for-reuse"></a>Aviso de cambio: Los códigos de autorización ya no estarán disponibles para su reutilización

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

A partir del 15 de noviembre de 2018, Azure AD dejará de aceptar los códigos de autenticación que se usaban anteriormente para las aplicaciones. Este cambio de seguridad ayuda a poner Azure AD en consonancia con la especificación de OAuth y se aplicará en los puntos de conexión v1 y v2.

Si la aplicación reutiliza códigos de autorización para obtener tokens para varios recursos, es recomendable que use el código para obtener un token de actualización y, a continuación, utilice este para adquirir tokens adicionales para otros recursos. Los códigos de autorización solo se pueden usar una vez, pero los tokens de actualización se pueden usar varias veces en varios recursos. Una aplicación que intente reutilizar un código de autenticación durante el flujo de código de OAuth obtendrá el error invalid_grant.

Para este y otros cambios relacionados con los protocolos, consulte [la lista completa de las novedades de la autenticación](../develop/reference-breaking-changes.md).

---

### <a name="converged-security-info-management-for-self-service-password-sspr-and-multi-factor-authentication-mfa"></a>Administración de información de seguridad convergida para restablecimiento de contraseña de autoservicio (SSPR) y Multi-Factor Authentication (MFA)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** SSPR Autenticación de usuarios

Esta nueva característica ayuda a los usuarios a administrar su información de seguridad (por ejemplo, número de teléfono, aplicación móvil, etc.) para SSPR y MFA en una sola ubicación y una misma experiencia, al contrario que sucedía anteriormente cuando se realizaba en dos ubicaciones diferentes.

Esta experiencia convergente también funciona para los usuarios que utilizan MFA o SSPR. Además, aunque la organización no aplique el registro de MFA o de SSPR, los usuarios pueden registrar todos los métodos de información de seguridad que permita la organización desde el portal Mis aplicaciones.

Se trata de una versión preliminar pública opcional. Los administradores pueden activar la nueva experiencia (si lo desean) para un grupo de usuarios seleccionado o para todos los usuarios de un inquilino. Para más información sobre la experiencia convergente, consulte el [blog de experiencia convergente](https://cloudblogs.microsoft.com/enterprisemobility/2018/08/06/mfa-and-sspr-updates-now-in-public-preview/)

---

### <a name="new-http-only-cookies-setting-in-azure-ad-application-proxy-apps"></a>Nueva configuración de las cookies solo HTTP en las aplicaciones de Azure AD Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Hay un nuevo valor llamado, **HTTP-Only Cookies** (Cookies solo HTTP) en las aplicaciones de Application Proxy. Este valor ayuda a proporcionar un nivel extra de seguridad incluyendo la marca HTTPOnly en el encabezado de la respuesta HTTP para las cookies de acceso a Application Proxy y las de la sesión, impidiendo el acceso a cookies procedentes de un script en el lado del cliente e impidiendo acciones como la copia o modificación de la cookie. Aunque esta marca no se ha usado anteriormente, las cookies se han cifrado y transmitido siempre a través de una conexión TLS para proteger frente a modificaciones no adecuadas.

Este valor no es compatible con aplicaciones que usan controles ActiveX, como Escritorio remoto. Si este es su caso, es aconsejable que desactive este valor.

Para más información acerca del valor de cookies solo HTTP, consulte [Publicación de aplicaciones mediante Azure AD Application Proxy](../app-proxy/application-proxy-add-on-premises-application.md).

---

### <a name="privileged-identity-management-pim-for-azure-resources-supports-management-group-resource-types"></a>Privileged Identity Management (PIM) para recursos de Azure es compatible con los tipos de recursos del grupo de administración

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

La configuración de la activación y asignación Just-In-Time se puede aplicar ahora a los tipos de recursos del grupo de administración, al igual que ya lo hace con las suscripciones, grupos de recursos y recursos (como máquinas virtuales, App Services, etc). Además, cualquier usuario con un rol que proporcione acceso de administrador para un grupo de administración puede detectar y administrar ese recurso en PIM.

Para más información acerca de PIM y los recursos de Azure, consulte [Detectar y administrar recursos de Azure mediante Privileged Identity Management](../privileged-identity-management/pim-resource-roles-discover-resources.md)

---

### <a name="application-access-preview-provides-faster-access-to-the-azure-ad-portal"></a>La característica de acceso a la aplicación (versión preliminar) proporciona un acceso más rápido al portal de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

En la actualidad, al activar un rol con PIM, puede que los permisos tarden más de 10 minutos en surtir efecto. Si opta por usar la característica de acceso a la aplicación, que se encuentra actualmente en versión preliminar pública, los administradores podrán acceder al portal de Azure AD tan pronto como se complete la solicitud de activación.

Actualmente, esta característica solo admite la experiencia del portal de Azure AD y los recursos de Azure. Para más información acerca de PIM y la característica de acceso a la aplicación, consulte [¿Qué es Azure AD Privileged Identity Management?](../privileged-identity-management/pim-configure.md)

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---august-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: agosto de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En agosto de 2018, hemos agregado estas 16 nuevas aplicaciones con compatibilidad con la federación para la galería de aplicaciones:

[Hornbill](../saas-apps/hornbill-tutorial.md), [Bridgeline Unbound](../saas-apps/bridgelineunbound-tutorial.md), [Sauce Labs - Mobile and Web Testing](../saas-apps/saucelabs-mobileandwebtesting-tutorial.md), [Meta Networks Connector](../saas-apps/metanetworksconnector-tutorial.md), [Way We Do](../saas-apps/waywedo-tutorial.md), [Spotinst](../saas-apps/spotinst-tutorial.md), [ProMaster (by Inlogik)](../saas-apps/promaster-tutorial.md), SchoolBooking, [4me](../saas-apps/4me-tutorial.md), [Dossier](../saas-apps/dossier-tutorial.md), [N2F - Expense reports](../saas-apps/n2f-expensereports-tutorial.md), [Comm100 Live Chat](../saas-apps/comm100livechat-tutorial.md), [SafeConnect](../saas-apps/safeconnect-tutorial.md), [ZenQMS](../saas-apps/zenqms-tutorial.md), [eLuminate](../saas-apps/eluminate-tutorial.md), [Dovetale](../saas-apps/dovetale-tutorial.md).

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="native-tableau-support-is-now-available-in-azure-ad-application-proxy"></a>Ahora ya está disponible la compatibilidad nativa con Tableau en Azure AD Application Proxy

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Gracias a la actualización del protocolo de concesión de código de OpenID Connect a OAuth 2.0 para nuestro protocolo de autenticación previa, ya no es necesario realizar ninguna configuración adicional para usar Tableau con Application Proxy. Este cambio de protocolo también ayuda a Application Proxy a mejorar su compatibilidad con aplicaciones más modernas mediante el uso exclusivo de redirecciones HTTP, que normalmente son compatibles con las etiquetas de JavaScript y HTML.

---

### <a name="new-support-to-add-google-as-an-identity-provider-for-b2b-guest-users-in-azure-active-directory-preview"></a>Nueva compatibilidad para agregar Google como proveedor de identidades de usuarios invitados de B2B en Azure Active Directory (versión preliminar)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C

Mediante la configuración de la federación con Google en su organización, puede permitir que usuarios invitados de Gmail inicien sesión en sus aplicaciones y recursos compartidos con su cuenta de Google existente sin tener que crear una cuenta Microsoft (MSA) personal o una cuenta de Azure AD.

Se trata de una versión preliminar pública opcional. Para más información sobre la federación con Google, consulte [Incorporación de Google como proveedor de identidades para los usuarios invitados de B2B](../external-identities/google-federation.md).

---

## <a name="july-2018"></a>Julio de 2018

### <a name="improvements-to-azure-active-directory-email-notifications"></a>Mejoras en las notificaciones de Azure Active Directory por correo electrónico

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Administración del ciclo de vida de la identidad

Los correos electrónicos de Azure Active Directory (Azure AD) ahora tienen un diseño actualizado, además de cambios en el nombre para mostrar del remitente y la dirección de correo electrónico del destinatario cuando los mensajes se envían desde los servicios siguientes:

- Revisiones de acceso de Azure AD
- Azure AD Connect Health
- Azure AD Identity Protection
- Administración de identidades con privilegios de Azure AD
- Aplicación empresarial: expiración de las notificaciones de certificado
- Aplicación empresarial: aprovisionamiento de las notificaciones de servicio

Las notificaciones de correo electrónico se enviarán desde la siguiente dirección de correo electrónico y nombre para mostrar:

- Dirección de correo electrónico: azure-noreply@microsoft.com
- Nombre para mostrar: Microsoft Azure

Para un ejemplo de algunos de los nuevos diseños de correo electrónico y más información, consulte [Email notifications in Azure AD PIM](../privileged-identity-management/pim-email-notifications.md) (Notificaciones por correo electrónico en Azure AD PIM).

---

### <a name="azure-ad-activity-logs-are-now-available-through-azure-monitor"></a>Los registros de actividad de Azure AD ahora están disponibles a través de Azure Monitor

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Los registros de actividad de Azure AD ahora están disponibles en la versión preliminar pública de Azure Monitor (servicio supervisión en toda la plataforma de Azure). Azure Monitor ofrece una retención a largo plazo e integración sin problemas, además de estas mejoras:

- Retención a largo plazo mediante el enrutamiento de los archivos de registro en su propia cuenta de almacenamiento de Azure.

- Integración SIEM perfecta, sin necesidad de escribir o mantener scripts personalizados.

- Integración sin problemas con sus propias soluciones personalizadas, herramientas de análisis o soluciones de administración de incidentes.

Para más información sobre estas nuevas funcionalidades, consulte nuestro blog [Azure AD activity logs in Azure Monitor diagnostics is now in public preview](https://cloudblogs.microsoft.com/enterprisemobility/2018/07/26/azure-ad-activity-logs-in-azure-monitor-diagnostics-now-in-public-preview/) (Los registros de actividad de Azure AD en diagnósticos de Azure Monitor están ahora en versión preliminar pública) y nuestra documentación, [Registros de actividad de Azure AD en Azure Monitor (versión preliminar)](../reports-monitoring/concept-activity-logs-azure-monitor.md).

---

### <a name="conditional-access-information-added-to-the-azure-ad-sign-ins-report"></a>Información de acceso condicional agregada al informe de inicios de sesión de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Seguridad y protección de la identidad

Esta actualización le permite ver qué directivas se evalúan cuando un usuario inicia sesión, junto con el resultado de la directiva. Además, el informe ahora incluye el tipo de aplicación cliente que el usuario utiliza, de manera que usted puede identificar el tráfico de protocolo heredado. Ahora también se pueden buscar entradas de informe para un identificador de correlación, que puede encontrarse en el mensaje de error de cara al usuario y puede usarse para identificar la solicitud de inicio de sesión coincidente y solucionar problemas en dicha solicitud.

---

### <a name="view-legacy-authentications-through-sign-ins-activity-logs"></a>Visualización de autenticaciones heredadas a través de los registros de actividad de inicios de sesión

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Con la introducción del campo **Aplicación cliente** en los registros de actividad de inicio de sesión, los clientes ya pueden ver los usuarios que utilizan autenticaciones heredadas. Los clientes podrán acceder a esta información mediante Microsoft Graph API de los registros de inicio de sesión o mediante los registros de actividad de inicio de sesión del portal Azure AD, donde se puede usar el control **Aplicación cliente** para filtrar por autenticaciones heredadas. Para más información, consulte la documentación.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---july-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: julio de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En julio de 2018, hemos agregado estas 16 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[Innovation Hub](../saas-apps/innovationhub-tutorial.md), [Leapsome](../saas-apps/leapsome-tutorial.md), [Certain Admin SSO](../saas-apps/certainadminsso-tutorial.md), PSUC Staging, [iPass SmartConnect](../saas-apps/ipasssmartconnect-tutorial.md), [Screencast-O-Matic](../saas-apps/screencast-tutorial.md), PowerSchool Unified Classroom, [Eli Onboarding](../saas-apps/elionboarding-tutorial.md), [Bomgar Remote Support](../saas-apps/bomgarremotesupport-tutorial.md), [Nimblex](../saas-apps/nimblex-tutorial.md), [Imagineer WebVision](../saas-apps/imagineerwebvision-tutorial.md), [Insight4GRC](../saas-apps/insight4grc-tutorial.md), [SecureW2 JoinNow Connector](../saas-apps/securejoinnow-tutorial.md), [Kanbanize](../saas-apps/kanbanize-tutorial.md), [SmartLPA](../saas-apps/smartlpa-tutorial.md) y [Skills Base](../saas-apps/skillsbase-tutorial.md).

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-user-provisioning-saas-app-integrations---july-2018"></a>Nuevas integraciones de aplicaciones de SaaS de aprovisionamiento de usuarios: julio de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Integración de terceros

Azure AD permite automatizar la creación, el mantenimiento y la eliminación de identidades de usuario en aplicaciones de SaaS, como Dropbox, Salesforce, ServiceNow, etc. En julio de 2018, hemos agregado compatibilidad con el aprovisionamiento de usuarios a las siguientes aplicaciones de la Galería de aplicaciones de Azure AD:

- [Cisco WebEx](../saas-apps/cisco-webex-provisioning-tutorial.md)

- [Bonusly](../saas-apps/bonusly-provisioning-tutorial.md)

Para una lista de todas las aplicaciones que admiten el aprovisionamiento de usuarios en la Galería de Azure AD, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md).

---

### <a name="connect-health-for-sync---an-easier-way-to-fix-orphaned-and-duplicate-attribute-sync-errors"></a>Connect Health para sincronización: una manera más sencilla de corregir errores de sincronización de atributos duplicados o huérfanas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Supervisión e informes

Azure AD Connect Health presenta la corrección de autoservicio para ayudarle a resaltar y corregir errores de sincronización. Esta característica soluciona los errores de sincronización de atributos duplicados y repara los objetos huérfanos desde Azure AD. Este diagnóstico tiene las siguientes ventajas:

- Reduce los errores de sincronización de atributos duplicado, proporcionando correcciones específicas

- Aplica una corrección para escenarios de Azure AD dedicados, resolviendo los errores en un solo paso dedicado

- No es necesaria ninguna actualización o configuración para activar y usar esta característica

Para más información, consulte [Diagnóstico y solución de errores de sincronización de atributos duplicados](../hybrid/how-to-connect-health-diagnose-sync-errors.md).

---

### <a name="visual-updates-to-the-azure-ad-and-msa-sign-in-experiences"></a>Actualizaciones de objetos visuales en Azure AD y experiencias de inicio de sesión de MSA

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Azure AD Autenticación de usuarios

Hemos actualizado la interfaz de usuario de la experiencia de inicio de sesión de los servicios en línea de Microsoft, como Office 365 y Azure. Este cambio hace que las pantallas estén menos saturadas y sean más sencillas. Para más información sobre este cambio, consulte el blog [Upcoming improvements to the Azure AD sign-in experience](https://cloudblogs.microsoft.com/enterprisemobility/2018/04/04/upcoming-improvements-to-the-azure-ad-sign-in-experience/) (Próximas mejoras en la experiencia de inicio de sesión de Azure AD).

---

### <a name="new-release-of-azure-ad-connect---july-2018"></a>Nueva versión de Azure AD Connect: julio de 2018

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Administración del ciclo de vida de la identidad

La versión más reciente de Azure AD Connect incluye:

- Correcciones de errores y actualizaciones de compatibilidad

- Disponibilidad general de la integración PingFederate

- Actualizaciones en el cliente más reciente de SQL 2012

Para más información sobre esta actualización, consulte [Azure AD Connect: historial de versiones](../hybrid/reference-connect-version-history.md)

---

### <a name="updates-to-the-terms-of-use-end-user-ui"></a>Actualizaciones en la interfaz de usuario del usuario final correspondiente a los términos de uso

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Términos de uso Gobernanza

Estamos actualizando la cadena de aceptación en la interfaz de usuario del usuario final correspondiente a las condiciones de uso.

**Texto actual.** Para acceder a los recursos de [nombreInquilino], debe aceptar las condiciones de uso.<br>**Nuevo texto.** Para acceder al recurso de [nombreInquilino], debe leer las condiciones de uso.

**Texto actual:** Si elige Aceptar, significa que acepta todas las condiciones de uso anteriores.<br>**Nuevo texto:** Haga clic en Aceptar para confirmar que ha leído y comprendido las condiciones de uso.

---

### <a name="pass-through-authentication-supports-legacy-protocols-and-applications"></a>La autenticación de paso a través es compatible con las aplicaciones y protocolos heredados

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

La autenticación de paso a través es compatible con las aplicaciones y los protocolos heredados. Ahora se admiten completamente las siguientes limitaciones:

- Inicios de sesión de usuario en las aplicaciones cliente de Office heredadas, Office 2010 y Office 2013, sin necesidad de autenticación moderna.

- Acceso a uso compartido del calendario e información de disponibilidad en entornos híbridos de Exchange solo en Office 2010.

- Inicios de sesión de usuarios en aplicaciones cliente de Skype Empresarial sin necesidad de autenticación moderna.

- Inicios de sesión de usuario en PowerShell 1.0.

- El Programa de inscripción de dispositivos de Apple (DEP de Apple), mediante el Asistente para configuración de iOS.

---

### <a name="converged-security-info-management-for-self-service-password-reset-and-multi-factor-authentication"></a>Administración de información de seguridad convergida para restablecimiento de contraseña de autoservicio y Multi-Factor Authentication

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** SSPR Autenticación de usuarios

Esta nueva característica permite a los usuarios administrar su información de seguridad (por ejemplo, número de teléfono, dirección de correo electrónico, aplicación móvil, etc.) para el restablecimiento de contraseña de autoservicio (SSPR) y Multi-Factor Authentication (MFA) en una sola experiencia. Los usuarios ya no tendrán que registrar la misma información de seguridad para SSPR y MFA en dos experiencias diferentes. Esta nueva experiencia también se aplica a los usuarios que tienen SSPR o MFA.

Si una organización no aplica el registro MFA o SSPR, los usuarios pueden registrar su información de seguridad a través del portal **Mis aplicaciones**. Desde allí, los usuarios pueden registrar los métodos habilitados para MFA o SSPR.

Se trata de una versión preliminar pública opcional. Los administradores pueden activar la nueva experiencia (si lo desean) para un grupo de usuarios seleccionado o para todos los usuarios en un inquilino.

---

### <a name="use-the-microsoft-authenticator-app-to-verify-your-identity-when-you-reset-your-password"></a>Uso de la aplicación Microsoft Authenticator para verificar su identidad cuando restablezca la contraseña

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** SSPR Autenticación de usuarios

Esta característica permite a los no administradores verificar su identidad al restablecer una contraseña mediante una notificación o un código de Microsoft Authenticator (o cualquier otra aplicación de autenticador). Una vez que los administradores activan este método de restablecimiento de contraseña de autoservicio, los usuarios que hayan registrado una aplicación móvil a través de aka.ms/mfasetup o aka.ms/setupsecurityinfo pueden usar su aplicación móvil como método de verificación al restablecer su contraseña.

La notificación de la aplicación móvil solo se puede activar como parte de una directiva que requiere dos métodos para restablecer la contraseña.

---

## <a name="june-2018"></a>Junio de 2018

### <a name="change-notice-security-fix-to-the-delegated-authorization-flow-for-apps-using-azure-ad-activity-logs-api"></a>Aviso de cambio: Revisión de seguridad para el flujo de autorización delegado para aplicaciones que usan la API de registro de actividad de Azure AD

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto**: Informes Supervisión e informes

Debido a nuestras medidas de seguridad más estrictas, hemos tenido que realizar un cambio en los permisos de las aplicaciones que usan un flujo de autorización delegado para acceder a la [API de registro de actividad de Azure AD](../reports-monitoring/concept-reporting-api.md). Este cambio se producirá el **26 de junio de 2018**.

Si alguna de sus aplicaciones usa la API de registro de actividad de Azure AD, siga estos pasos para asegurarse de que la aplicación no se interrumpa después de que se produzca el cambio.

**Para actualizar los permisos de la aplicación**

1. Inicie sesión en Azure Portal, seleccione **Azure Active Directory** y, a continuación, **Registros de aplicaciones**.
2. Seleccione la aplicación que usa la API de registro de actividad de Azure AD, seleccione **Configuración**, **Permisos necesarios** y, a continuación, seleccione la API de **Microsoft Azure Active Directory**.
3. En el área **Permisos delegados** de la hoja **Habilitar acceso**, active la casilla junto a **Leer datos de directorio** y seleccione **Guardar**.
4. Haga clic en **Conceder permisos** y, a continuación, haga clic en **Sí**.

    >[!Note]
    >Debe ser un administrador global para conceder permisos a la aplicación.

Para obtener más información, consulte el área [Conceder permisos](../reports-monitoring/howto-configure-prerequisites-for-reporting-api.md#grant-permissions) de los requisitos previos para obtener acceso al artículo de la API de generación de informes de Azure AD.

---

### <a name="configure-tls-settings-to-connect-to-azure-ad-services-for-pci-dss-compliance"></a>Configuración de las opciones de TLS para conectarse a servicios de Azure AD para el cumplimiento de PCI DSS

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** N/D Plataforma

La Seguridad de la capa de transporte (TLS) es un protocolo que proporciona privacidad e integridad de datos para la comunicación entre dos aplicaciones, y es el protocolo de seguridad más implementado hoy en día.

El [PCI Security Standards Council](https://www.pcisecuritystandards.org/) (Consejo de estándares de seguridad de PCI) ha determinado que las versiones anteriores de TLS y Capa de sockets seguros (SSL) deben deshabilitarse para habilitar protocolos de aplicaciones nuevos y más seguros, con el inicio del cumplimiento a partir del **30 de junio 2018**. Este cambio significa que, si se conecta a servicios de Azure AD y es necesario el cumplimiento con PCI DSS, deberá deshabilitar TLS 1.0. Hay varias versiones de TLS disponibles, pero TLS 1.2 es la más reciente para servicios de Azure Active Directory. Se recomienda encarecidamente moverse directamente a TLS 1.2 para las combinaciones de cliente/servidor y explorador/servidor.

Es posible que los exploradores obsoletos no admitan las versiones más recientes de TLS, como TLS 1.2. Para ver qué versiones de TLS son compatibles con el explorador, vaya al sitio de [Qualys SSL Labs](https://www.ssllabs.com/) y haga clic en **Test your browser** (Probar el explorador). Le recomendamos que actualice a la versión más reciente del explorador web y, si es posible,que habilite solo TLS 1.2.

**Para habilitar TLS 1.2 por explorador**

- **Microsoft Edge e Internet Explorer (ambos se establecen mediante Internet Explorer)**

    1. Abra Internet Explorer, seleccione **Herramientas** > **Opciones de Internet** > **Opciones avanzadas**.
    2. En el área de **Seguridad**, seleccione **use TLS 1.2** (usar TLS 1.2) y, a continuación, seleccione **Aceptar**.
    3. Cierre todas las ventanas del explorador y reinicie Internet Explorer.

- **Google Chrome**

    1. Abra Google Chrome, escriba *chrome://settings/* en la barra de direcciones y presione **ENTRAR**.
    2. Expanda la **Configuración avanzada**, vaya al área de **Sistema** y seleccione **Abrir la configuración de proxy**.
    3. En el cuadro **Propiedades: Internet**, seleccione la pestaña **Opciones avanzadas**, vaya al área de **Seguridad**, seleccione **Usar TLS 1.2** y, a continuación, seleccione  **Aceptar**.
    4. Cierre todas las ventanas del explorador y reinicie Google Chrome.

- **Mozilla Firefox**

    1. Abra Firefox, escriba *about:config* en la barra de direcciones y, a continuación, presione **ENTRAR**.
    2. Busque el término *TLS* y, a continuación, seleccione la entrada **security.tls.version.max**.
    3. Establezca el valor en **3** para forzar al explorador a usar hasta la versión TLS 1.2 y, a continuación, seleccione **Aceptar**.

        >[!NOTE]
        >La versión de Firefox 60.0 admite TLS 1.3, por lo que también puede establecer el valor de security.tls.version.max en **4**.

    4. Cierre todas las ventanas del explorador y reinicie Mozilla Firefox.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---june-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: junio de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En junio de 2018, hemos agregado estas 15 nuevas aplicaciones con compatibilidad con la federación para la galería de aplicaciones:

[Skytap](../saas-apps/skytap-tutorial.md), [Settling music](../saas-apps/settlingmusic-tutorial.md), [aplicación de LOB con el token SAML 1.1 habilitado](../saas-apps/saml-tutorial.md), [Supermood](../saas-apps/supermood-tutorial.md), [Autotask](../saas-apps/autotaskendpointbackup-tutorial.md), [Endpoint Backup](../saas-apps/autotaskendpointbackup-tutorial.md), [Skyhigh Networks](../saas-apps/skyhighnetworks-tutorial.md), Smartway2, [TonicDM](../saas-apps/tonicdm-tutorial.md), [Moconavi](../saas-apps/moconavi-tutorial.md), [Zoho One](../saas-apps/zohoone-tutorial.md), [SharePoint local](../saas-apps/sharepoint-on-premises-tutorial.md), [ForeSee CX Suite](../saas-apps/foreseecxsuite-tutorial.md), [Vidyard](../saas-apps/vidyard-tutorial.md) y [ChronicX](../saas-apps/chronicx-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md). Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="azure-ad-password-protection-is-available-in-public-preview"></a>La Protección con contraseña de Azure AD está disponible en versión preliminar pública.

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Autenticación de usuarios

Use la Protección con contraseña de Azure AD para ayudar a eliminar de su entorno las contraseñas que se pueden averiguar fácilmente. Al eliminar estas contraseñas, ayuda a reducir el riesgo de un tipo de ataque de difusión de contraseña.

En concreto, la Protección con contraseña de Azure AD:

- Le ayuda a proteger las cuentas de la organización en Azure AD y Windows Server Active Directory (AD).
- Impide que los usuarios usen las contraseñas de una lista que incluye más de 500 de las contraseñas utilizadas con más frecuencia y más de un millón de variaciones de sustitución de caracteres de esas contraseñas.
- Le ayuda a administrar la Protección con contraseña de Azure AD desde una única ubicación en el portal de Azure AD, tanto para Azure AD como para Windows Server AD local.

Para obtener más información acerca de la Protección con contraseña de Azure AD, consulte [Elimine las contraseñas incorrectas de su organización](../authentication/concept-password-ban-bad.md).

---

### <a name="new-all-guests-conditional-access-policy-template-created-during-terms-of-use-creation"></a>Nueva plantilla "todos los invitados" de la directiva de acceso condicional creada durante la creación de los términos de uso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Gobernanza

Durante la creación de los términos de uso, también se ha creado una nueva plantilla de la directiva de acceso condicional para "todos los invitados" y "todas las aplicaciones". Esta nueva plantilla de la directiva se aplica a los Términos de uso recién creados, lo que facilita el proceso de cumplimiento y creación para los invitados.

Para obtener más información, consulte [Característica Azure Active Directory Terms of Use](../conditional-access/terms-of-use.md).

---

### <a name="new-custom-conditional-access-policy-template-created-during-terms-of-use-creation"></a>Nueva plantilla "personalizado" de la directiva de acceso condicional creada durante la creación de los términos de uso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Gobernanza

Durante la creación de los términos de uso, también se ha creado una nueva plantilla "personalizada" de la directiva de acceso condicional. Esta nueva plantilla de la directiva le permite crear los términos de uso y luego pasar inmediatamente a la hoja de creación de la directiva de acceso condicional, sin necesidad de navegar por el portal de forma manual.

Para obtener más información, consulte [Característica Azure Active Directory Terms of Use](../conditional-access/terms-of-use.md).

---

### <a name="new-and-comprehensive-guidance-about-deploying-azure-ad-multi-factor-authentication"></a>Guía nueva y completa acerca de cómo implementar Azure AD Multi-Factor Authentication

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Otros Seguridad y protección de la identidad

Hemos lanzado la nueva guía paso a paso acerca de cómo implementar Azure AD Multi-Factor Authentication (MFA) en su organización.

Para ver la guía de implementación de MFA, vaya al repositorio [Guías de implementación de identidad](./active-directory-deployment-plans.md) en GitHub. Para enviar comentarios acerca de las guías de implementación, use el [formulario Comentarios del plan de implementación](https://aka.ms/deploymentplanfeedback). Si tiene alguna pregunta acerca de las guías de implementación, póngase en contacto con nosotros en [IDGitDeploy](mailto:idgitdeploy@microsoft.com).

---

### <a name="azure-ad-delegated-app-management-roles-are-in-public-preview"></a>Los roles de administración de aplicaciones delegadas de Azure AD están en versión preliminar pública.

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Control de acceso

Ahora los administradores pueden delegar tareas de administración de aplicaciones sin asignar el rol de administrador global. Los nuevos roles y funcionalidades son:

- **Nuevos roles de administrador de Azure AD estándar:**

    - **Administrador de aplicaciones** Concede la capacidad de administrar todos los aspectos de todas las aplicaciones, incluidos el registro, la configuración de SSO, las asignaciones de aplicaciones y licencias, la configuración del proxy de aplicación y el consentimiento (excepto para los recursos de Azure AD).

    - **Administrador de aplicaciones en la nube** Concede todas las capacidades de administrador de aplicaciones, excepto para el proxy de aplicación porque no proporciona acceso local.

    - **Desarrollador de aplicaciones** Concede permisos para crear registros de aplicaciones, incluso si está desactivada la opción **allow users to register apps** (permitir a los usuarios registrar aplicaciones).

- **Propiedad (configurar el registro por aplicación y la aplicación por empresa, similar al proceso de la propiedad de grupo:**

    - **Propietario de registro de aplicaciones** Concede la capacidad de administrar todos los aspectos del registro de aplicaciones en propiedad, incluidos el manifiesto de la aplicación y la adición de más propietarios.

    - **Propietario de aplicaciones empresariales** Concede la capacidad de administrar muchos aspectos de las aplicaciones empresariales en propiedad, incluidos el consentimiento, la configuración de SSO y las asignaciones de aplicaciones (excepto para los recursos de Azure AD).

Para obtener más información acerca de la versión preliminar pública, consulte el blog [Azure AD delegated application management roles are in public preview!](https://cloudblogs.microsoft.com/enterprisemobility/2018/06/13/hallelujah-azure-ad-delegated-application-management-roles-are-in-public-preview/) (Los roles de administración de aplicaciones delegadas de Azure AD están en versión preliminar pública). . Para obtener más información acerca de los roles y permisos, consulte [Assigning administrator roles in Azure Active Directory](../roles/permissions-reference.md) (Asignación de roles de administrador en Azure Active Directory).

---

## <a name="may-2018"></a>Mayo de 2018

### <a name="expressroute-support-changes"></a>Cambios de soporte técnico de ExpressRoute

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Plataforma

Las ofertas de software como servicio, como Azure Active Directory (Azure AD) están diseñadas para funcionar mejor a través de Internet, sin necesidad de ExpressRoute ni otros túneles VPN privados. Por este motivo, en **1 de agosto de 2018**, dejaremos de dar soporte técnico a ExpressRoute para los servicios de Azure AD que usen un emparejamiento público de Azure y a las comunidades de Azure que usen emparejamientos de Microsoft. Los servicios afectados por este cambio podrían observar que el tráfico de Azure AD gradualmente cambia de ExpressRoute a Internet.

Aunque vamos a cambiar el soporte, también sabemos que aún hay situaciones en las que tendrá que utilizar un conjunto de circuitos dedicado para el tráfico de autenticación. Por este motivo, Azure AD seguirá admitiendo restricciones de intervalos de IP por inquilino utilizando ExpressRoute y los servicios que ya tienen emparejamiento de Microsoft con la comunidad "Otros servicios en línea de Office 365". Si los servicios se ven afectados pero requieren ExpressRoute, debe hacer lo siguiente:

- **Si está en un emparejamiento público de Azure.** Cambie a un emparejamiento de Microsoft y suscríbase a la comunidad **Otros servicios en línea de Office 365 (12076:5100)** . Para más información sobre cómo cambiar de emparejamiento público de Azure a emparejamiento de Microsoft, consulte el artículo [Cambiar un emparejamiento público a emparejamiento de Microsoft](../../expressroute/how-to-move-peering.md).

- **Si está en un emparejamiento de Microsoft.** Suscríbase a la comunidad **Otros servicio en línea de Office 365 (12076:5100)** . Para más información sobre los requisitos de enrutamiento, consulte la sección [Soporte técnico para las comunidades de BGP](../../expressroute/expressroute-routing.md#bgp) en el artículo de requisitos de enrutamiento de ExpressRoute.

Si debe continuar usando circuitos dedicados, deberá hablar con el equipo de su cuenta Microsoft acerca de cómo obtener autorización para utilizar la comunidad **Otros servicios en línea de Office 365 (12076:5100)** . El comité de revisión administrado por MS Office comprobará si necesita los circuitos y se asegurará de que comprende las implicaciones técnicas de su conservación. Las suscripciones no autorizadas que intenten crear filtros de ruta para Office 365 recibirán un mensaje de error.

---

### <a name="microsoft-graph-apis-for-administrative-scenarios-for-tou"></a>Microsoft Graph API para escenarios administrativos de términos de uso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Experiencia para el desarrollador

Hemos agregado Microsoft Graph API para el funcionamiento de la administración de los términos de uso de Azure AD. Puede crear, actualizar y eliminar el objeto de términos de uso.

---

### <a name="add-azure-ad-multi-tenant-endpoint-as-an-identity-provider-in-azure-ad-b2c"></a>Agregar punto de conexión multiinquilino de Azure AD como proveedor de identidades en Azure AD B2C

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Con las directivas personalizadas, ahora puede agregar el punto de conexión común de Azure AD como proveedor de identidades en Azure AD B2C. Esto le permite tener un único punto de entrada para todos los usuarios de Azure AD que inician sesión en sus aplicaciones. Para más información, consulte [Azure Active Directory B2C: Permitir que los usuarios inicien sesión en un proveedor de identidades de Azure AD multiinquilino mediante directivas personalizadas](../../active-directory-b2c/identity-provider-azure-ad-multi-tenant.md).

---

### <a name="use-internal-urls-to-access-apps-from-anywhere-with-our-my-apps-sign-in-extension-and-the-azure-ad-application-proxy"></a>Uso de direcciones URL internas para acceder a aplicaciones desde cualquier lugar con My Apps Sign-in Extension y Azure AD Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones SSO

Los usuarios ya pueden acceder a las aplicaciones a través de direcciones URL internas, incluso desde fuera de la red corporativa, mediante My Apps Sign-in Extension para Azure AD. Esto funcionará con todas las aplicaciones que se ha publicado con Azure Active Directory Application Proxy, en cualquier explorador que también tenga instalada la extensión de explorador del Panel de acceso. La funcionalidad de redireccionamiento de direcciones URL se habilita automáticamente una vez que un usuario inicia sesión en la extensión. La extensión se puede descargar en [Microsoft Edge](https://go.microsoft.com/fwlink/?linkid=845176), [Chrome](https://go.microsoft.com/fwlink/?linkid=866367) y [Firefox](https://go.microsoft.com/fwlink/?linkid=866366).

---

### <a name="azure-active-directory---data-in-europe-for-europe-customers"></a>Azure Active Directory: datos en Europa para los clientes europeos

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Otros GoLocal

Los clientes de Europa requieren que sus datos se queden en Europa y no se repliquen fuera de los centros de datos europeos para proteger la privacidad y cumplir la legislación europea. En este [artículo](./active-directory-data-storage-eu.md) se proporcionan detalles concretos no solo acerca de qué información de identidad se almacenará en Europa, sino también acerca de la que se almacenará fuera de los centros de datos europeos.

---

### <a name="new-user-provisioning-saas-app-integrations---may-2018"></a>Nuevas integraciones de aplicaciones de SaaS de aprovisionamiento de usuarios: mayo de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Integración de terceros

Azure AD permite automatizar la creación, el mantenimiento y la eliminación de identidades de usuario en aplicaciones de SaaS, como Dropbox, Salesforce, ServiceNow, etc. En mayo de 2018, hemos agregado compatibilidad con el aprovisionamiento de usuarios a las siguientes aplicaciones de la Galería de aplicaciones de Azure AD:

- [BlueJeans](../saas-apps/bluejeans-provisioning-tutorial.md)

- [Cornerstone OnDemand](../saas-apps/cornerstone-ondemand-provisioning-tutorial.md)

- [Zendesk](../saas-apps/zendesk-provisioning-tutorial.md)

Para obtener una lista de todas las aplicaciones que admiten el aprovisionamiento de usuarios en la Galería de Azure AD, consulte [https://aka.ms/appstutorial](../saas-apps/tutorial-list.md).

---

### <a name="azure-ad-access-reviews-of-groups-and-app-access-now-provides-recurring-reviews"></a>Ahora, las revisiones de acceso de Azure AD de grupos y aplicaciones proporcionan revisiones periódicas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Revisiones de acceso Gobernanza

La revisión de acceso de grupos y aplicaciones ya está disponible de forma generalizada como parte de Azure AD Premium P2.  Los administradores podrán configurar las revisiones de acceso de los miembros de un grupo y las asignaciones de aplicaciones para que se repitan automáticamente a intervalos periódicos; por ejemplo, cada mes o cada trimestre.

---

### <a name="azure-ad-activity-logs-sign-ins-and-audit-are-now-available-through-ms-graph"></a>Los registros de actividad de Azure AD (inicios de sesión y auditoría) ahora están disponibles a través de MS Graph

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Los registros de actividad de Azure AD, que incluyen los registros inicios de sesión y de auditorías, ahora están disponibles a través de Microsoft Graph API. Para acceder a dichos registros se han expuesto dos puntos de conexión a través de Microsoft Graph API. Consulte nuestros [documentos](../reports-monitoring/concept-reporting-api.md) para ver cómo acceder mediante programación a las API de informes de Azure AD.

---

### <a name="improvements-to-the-b2b-redemption-experience-and-leave-an-org"></a>Mejoras en la experiencia de canje de invitaciones B2B y en el abandono de una organización

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C

**Canje justo a tiempo:** una vez que se comparte un recurso con un usuario invitado con la API B2B, no es preciso enviar un correo electrónico de invitación especial. En la mayoría de los casos, el usuario invitado puede acceder al recurso y pasar por la experiencia de canje justo a tiempo. Se acabaron los problemas por mensajes de correo electrónico perdidos. Se acabó preguntar a los usuarios invitados: "¿Ha hecho clic en el vínculo de canje que le envió el sistema?". Esto significa que una vez que la SPO utiliza el administrador de invitaciones, los elementos adjuntos de la nube pueden tener la misma dirección URL canónica para todos los usuarios, tanto internos como externos, con cualquier estado de canje.

**Experiencia de canje moderna:** se acabaron las páginas de aterrizaje para el canje con pantalla dividida. Los usuarios verán un consentimiento moderno con la declaración de privacidad de la organización que realiza la invitación, igual que en las aplicaciones de terceros.

**Los usuarios invitados pueden abandonar la organización:** Cuando termina la relación de un usuario con una organización, este puede salir de ella sin ayuda de nadie. No será preciso pedir al administrador de la organización que le invitó que lo "elimine" ni necesitará generar incidencias de soporte técnico.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---may-2018"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD: mayo de 2018

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En mayo de 2018, hemos agregado estas 18 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[AwardSpring](../saas-apps/awardspring-tutorial.md), Infogix Data3Sixty Govern, [Yodeck](../saas-apps/infogix-tutorial.md), [Jamf Pro](../saas-apps/jamfprosamlconnector-tutorial.md), [KnowledgeOwl](../saas-apps/knowledgeowl-tutorial.md), [Envi MMIS](../saas-apps/envimmis-tutorial.md), [LaunchDarkly](../saas-apps/launchdarkly-tutorial.md), [Adobe Captivate Prime](../saas-apps/adobecaptivateprime-tutorial.md), [Montage Online](../saas-apps/montageonline-tutorial.md), [まなびポケット](../saas-apps/manabipocket-tutorial.md), OpenReel, [Arc Publishing - SSO](../saas-apps/arc-tutorial.md), [PlanGrid](../saas-apps/plangrid-tutorial.md), [iWellnessNow](../saas-apps/iwellnessnow-tutorial.md), [Proxyclick](../saas-apps/proxyclick-tutorial.md), [Riskware](../saas-apps/riskware-tutorial.md), [Flock](../saas-apps/flock-tutorial.md), [Reviewsnap](../saas-apps/reviewsnap-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md).

Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-step-by-step-deployment-guides-for-azure-active-directory"></a>Nuevas guías de implementación paso a paso para Azure Active Directory

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Otros Directorio

Novedad: guía paso a paso sobre cómo implementar Azure Active Directory (Azure AD), incluido el autoservicio de restablecimiento de contraseña (SSPR), el inicio de sesión único (SSO), el acceso condicional (CA), el proxy de aplicación, el aprovisionamiento de usuarios, y los Servicios de federación de Active Directory (AD FS) en Autenticación de paso a través (PTA) y ADFS para realizar la sincronización de hash de contraseña (PBS).

Para ver las guías de implementación, vaya al repositorio [Guías de implementación de identidad](./active-directory-deployment-plans.md) en GitHub. Para enviar comentarios acerca de las guías de implementación, use el [formulario Comentarios del plan de implementación](https://aka.ms/deploymentplanfeedback). Si tiene alguna pregunta acerca de las guías de implementación, póngase en contacto con nosotros en [IDGitDeploy](mailto:idgitdeploy@microsoft.com).

---

### <a name="enterprise-applications-search---load-more-apps"></a>Búsqueda de aplicaciones empresariales: cargar más aplicaciones

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

¿No consigue encontrar aplicaciones o entidades de servicio? Hemos agregado la posibilidad de cargar más aplicaciones en la lista de todas las aplicaciones empresariales. De manera predeterminada, se muestran 20 aplicaciones. Ahora puede hacer clic en **Cargar más** y ver otras aplicaciones.

---

### <a name="the-may-release-of-aadconnect-contains-a-public-preview-of-the-integration-with-pingfederate-important-security-updates-many-bug-fixes-and-new-great-new-troubleshooting-tools"></a>La versión de mayo de AADConnect contiene una versión preliminar pública de la integración con PingFederate, actualizaciones de seguridad importantes, muchas correcciones de errores y nuevas y magníficas herramientas para la solución de problemas.

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto**: Administración del ciclo de vida de la identidad

La versión de mayo de AADConnect contiene una versión preliminar pública de la integración con PingFederate, actualizaciones de seguridad importantes, muchas correcciones de errores y nuevas y magníficas herramientas para la solución de problemas. Puede encontrar las notas de la versión [aquí](../hybrid/reference-connect-version-history.md).

---

### <a name="azure-ad-access-reviews-auto-apply"></a>Revisiones de acceso de Azure AD: aplicación automática

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Revisiones de acceso Gobernanza

Las revisiones de acceso de grupos y aplicaciones ahora están disponibles de forma generalizada como parte de Azure AD Premium P2. Un administrador puede realizar la configuración necesaria para aplicar automáticamente los cambios del revisor a un grupo o aplicación cuando se complete la revisión de acceso. El administrador también puede especificar lo que ocurre al acceso continuado del usuario si los revisores no responden: quitar el acceso, mantener el acceso o seguir las recomendaciones del sistema.

---

### <a name="id-tokens-can-no-longer-be-returned-using-the-query-response_mode-for-new-apps"></a>Los tokens de identificador no se pueden devolver mediante el valor de response_mode de una consulta en las nuevas aplicaciones.

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Las aplicaciones creadas a partir del 25 de abril de 2018 ya no podrán solicitar un valor de **id_token** mediante el argumento response_mode de **query**.  De esta forma, Azure AD se alinea con las especificaciones de OIDC y se reduce la superficie de ataque de sus aplicaciones.  No se impide que las aplicaciones creadas antes del 25 de abril de 2018 utilicen el argumento response_mode de **query** con un valor de response_type de **id_token**.  El error que se devuelve al solicitar un id_token de Azure AD es **AADSTS70007: "query" no es un valor de "response_mode" que se admita al solicitar un token**.

Los valores de response_mode **fragment** y **form_post** siguen funcionando (al crear nuevos objetos de aplicación [por ejemplo, para el uso de proxy de aplicación]). Asegúrese de usar uno de estos valores de response_mode antes de que creen una aplicación nueva.

---

## <a name="april-2018"></a>Abril de 2018

### <a name="azure-ad-b2c-access-token-are-ga"></a>Los tokens de acceso de Azure AD B2C son GA

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Ya puede acceder a las API web que protege Azure AD B2C gracias a los tokens de acceso. Esta característica está pasando de la versión preliminar pública a la versión de disponibilidad general. A parte de otras mejoras menores, también se ha mejorado la experiencia de interfaz de usuario para configurar las aplicaciones de Azure AD B2C y las API web.

Para más información, consulte [Azure AD B2C: Solicitud de tokens de acceso](../../active-directory-b2c/access-tokens.md).

---

### <a name="test-single-sign-on-configuration-for-saml-based-applications"></a>Probar la configuración del inicio de sesión único para aplicaciones basadas en SAML

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Al configurar las aplicaciones SSO basadas en SAML, puede probar la integración en la página de configuración. Si se produce un error durante el inicio de sesión, puede proporcionar dicho error en la experiencia de prueba y Azure AD le proporcionará los pasos para resolver ese problema.

Para más información, consulte:

- [Configuración del inicio de sesión único en aplicaciones que no están en la Galería de aplicaciones de Azure Active Directory](../manage-apps/view-applications-portal.md)
- [Cómo depurar el inicio de sesión único basado en SAML en aplicaciones de Azure Active Directory](../manage-apps/debug-saml-sso-issues.md)

---

### <a name="azure-ad-terms-of-use-now-has-per-user-reporting"></a>Los términos de uso de Azure AD ahora tienen informes por usuario

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Los administradores pueden seleccionar una versión determinada de los Términos de uso y ver qué usuarios han dado su consentimiento y en qué fecha y hora.

Para obtener más información, consulte [Azure AD terms of use feature](../conditional-access/terms-of-use.md) (Característica de términos de uso de Azure AD).

---

### <a name="azure-ad-connect-health-risky-ip-for-ad-fs-extranet-lockout-protection"></a>Azure AD Connect Health: Direcciones IP de riesgo para la protección de bloqueo de extranet de AD FS

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Otros Supervisión e informes

Connect Health ahora tiene la capacidad de detectar direcciones IP que superan un umbral de inicios de sesión de U/P erróneos por hora o por día. Estas son las funcionalidades que proporciona esta característica:

- Un informe completo que muestra la dirección IP y el número de inicios de sesión erróneos generados por hora o por día y con un umbral personalizable.
- Alertas basadas en el correo electrónico que muestran el momento en que una dirección IP específica excede el umbral de inicios de sesión de U/P erróneos por hora o por día.
- Una opción de descarga para realizar un análisis detallado de los datos.

Para obtener más información, consulte [Informe de direcciones IP de riesgo](../hybrid/how-to-connect-health-adfs.md).

---

### <a name="easy-app-config-with-metadata-file-or-url"></a>Configuración sencilla de aplicaciones mediante una dirección URL o un archivo de metadatos

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

En la página de aplicaciones Enterprise, los administradores pueden cargar un archivo de metadatos SAML para configurar el inicio de sesión basado en SAML tanto de las aplicaciones de la galería de Azure AD, como de las que no pertenecen a la galería.

Asimismo, puede usar la dirección URL de metadatos de la federación de aplicaciones de Azure AD para configurar el SSO con la aplicación de destino.

Para obtener más información, consulte [Configuring single sign-on to applications that are not in the Azure Active Directory application gallery](../manage-apps/view-applications-portal.md) (Configurar el inicio de sesión único de las aplicaciones que no forman parte de la galería de aplicaciones de Azure Active Directory).

---

### <a name="azure-ad-terms-of-use-now-generally-available"></a>Los Términos de uso de Azure AD están disponibles de forma general

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo


Los términos de uso de Azure AD han dejado de estar en versión preliminar pública y ahora están disponibles de forma general.

Para obtener más información, consulte [Azure AD terms of use feature](../conditional-access/terms-of-use.md) (Característica de términos de uso de Azure AD).

---

### <a name="allow-or-block-invitations-to-b2b-users-from-specific-organizations"></a>Permitir o bloquear invitaciones a usuarios B2B procedentes de determinadas organizaciones

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C


Ya puede especificar con qué organizaciones de socio quiere compartir contenido y colaborar en Azure AD B2B Collaboration. Para hacer esto, puede elegir crear una lista de dominios específicos para otorgar permiso o denegarlo. Cuando se bloquea un dominio mediante estas funcionalidades, los empleados ya no pueden enviar invitaciones a los usuarios de ese dominio.

Esto le ayudará a controlar el acceso a los recursos, a la vez que ofrece una experiencia fluida a los usuarios aprobados.

La característica Colaboración B2B está disponible para todos los clientes de Azure Active Directory y se puede usar con las características de Azure AD Premium, como el acceso condicional y la protección de identidad, para saber al detalle cómo y cuándo inician sesión y obtienen acceso los usuarios comerciales externos.

Para obtener más información, consulte [Allow or block invitations to B2B users from specific organizations](../external-identities/allow-deny-list.md) (Permitir o bloquear invitaciones a usuarios de B2B procedentes de determinadas organizaciones).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Nuevas aplicaciones federadas disponibles en la galería de aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En abril de 2018, hemos agregado estas 13 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

Criterion HCM, [FiscalNote](../saas-apps/fiscalnote-tutorial.md), [Secret Server (On-Premises)](../saas-apps/secretserver-on-premises-tutorial.md), [Dynamic Signal](../saas-apps/dynamicsignal-tutorial.md), [mindWireless](../saas-apps/mindwireless-tutorial.md), [OrgChart Now](../saas-apps/orgchartnow-tutorial.md), [Ziflow](../saas-apps/ziflow-tutorial.md), [AppNeta Performance Monitor](../saas-apps/appneta-tutorial.md), [Elium](../saas-apps/elium-tutorial.md), [Fluxx Labs](../saas-apps/fluxxlabs-tutorial.md), [Cisco Cloud](../saas-apps/ciscocloud-tutorial.md), Shelf, [SafetyNet](../saas-apps/safetynet-tutorial.md)

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md).

Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="grant-b2b-users-in-azure-ad-access-to-your-on-premises-applications-public-preview"></a>Conceder a los usuarios de B2B en Azure AD acceso a las aplicaciones locales (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2B B2B/B2C

Las organizaciones que usan las funcionalidades de colaboración B2B de Azure Active Directory (Azure AD) para invitar a los usuarios invitados de organizaciones asociadas a su instancia de Azure AD, ahora pueden proporcionar a estos usuarios B2B acceso a las aplicaciones locales. Estas aplicaciones locales pueden usar la autenticación basada en SAML o la autenticación de Windows integrada (IWA) con la delegación limitada de kerberos (KCD).

Para obtener más información, consulte [Conceder a los usuarios de B2B en Azure AD acceso a las aplicaciones locales](../external-identities/hybrid-cloud-to-on-premises.md).

---

### <a name="get-sso-integration-tutorials-from-the-azure-marketplace"></a>Obtener tutoriales de integración de SSO en Azure Marketplace

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Otros Integración de terceros

Si una aplicación incluida en [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps?page=1) admite el proceso de inicio de sesión único basado en SAML, al hacer clic en **Obtener ahora**  se proporciona el tutorial de integración asociado a esa aplicación.

---

### <a name="faster-performance-of-azure-ad-automatic-user-provisioning-to-saas-applications"></a>Rendimiento más rápido del aprovisionamiento de usuarios automático de Azure AD para aplicaciones SaaS

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Integración de terceros

Anteriormente, los clientes que utilizaban los conectores de aprovisionamiento de usuarios de Azure Active Directory para aplicaciones de SaaS (por ejemplo, Salesforce, ServiceNow y Box) podían experimentar un rendimiento lento si sus inquilinos de Azure AD contenían más de 100 000 usuarios y grupos combinados, y si usaban asignaciones de usuarios y grupos para determinar qué usuarios debían aprovisionarse.

El 2 de abril de 2018 se implementaron mejoras de rendimiento significativas en el servicio de aprovisionamiento de Azure AD que reducen en gran medida la cantidad de tiempo necesario para realizar las sincronizaciones iniciales entre Azure Active Directory y las aplicaciones de SaaS de destino.

Como resultado, ahora se completan en cuestión de minutos u horas sincronizaciones iniciales que muchos clientes realizaban en sus aplicaciones y que antes tardaban días en completarse o que nunca llegaban a completarse.

Para obtener más información, consulte [¿Qué ocurre durante el aprovisionamiento?](../..//active-directory/app-provisioning/how-provisioning-works.md)

---

### <a name="self-service-password-reset-from-windows-10-lock-screen-for-hybrid-azure-ad-joined-machines"></a>Restablecimiento de contraseña de autoservicio desde la pantalla de bloqueo de Windows 10 para máquinas híbridas unidas a Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Autenticación de usuarios

Hemos actualizado la característica SSPR de Windows 10 para incluir soporte técnico para máquinas híbridas que se hayan unido a Azure AD. Esta característica está disponible en Windows 10 RS4 y permite a los usuarios restablecer la contraseña en la pantalla de bloqueo de una máquina Windows 10. Los usuarios que están habilitados y registrados para poder realizar el restablecimiento de contraseña de autoservicio pueden utilizar esta característica.

Para obtener más información, consulte [Azure AD password reset from the login screen](../authentication/howto-sspr-windows.md) (Restablecer la contraseña de Azure AD desde la pantalla de inicio de sesión).

---

## <a name="march-2018"></a>Marzo de 2018

### <a name="certificate-expire-notification"></a>Notificación de expiración de certificado

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Aplicaciones empresariales SSO

Azure AD envía una notificación cuando un certificado de una aplicación incluida o no en una galería está a punto de expirar.

Algunos usuarios no recibían notificaciones sobre aplicaciones empresariales configuradas con el inicio de sesión único basado en SAML. Este problema se ha resuelto. Azure AD envía una notificación para los certificados que expiran en 7, 30 y 60 días. Este evento puede verse en los registros de auditoría.

Para más información, consulte:

- [Administrar certificados para inicio de sesión único federado en Azure Active Directory](../manage-apps/manage-certificates-for-federated-single-sign-on.md)
- [Informes de actividad de auditoría en el portal de Azure Active Directory](../reports-monitoring/concept-audit-logs.md)

---

### <a name="twitter-and-github-identity-providers-in-azure-ad-b2c"></a>Proveedores de identidades de Twitter y GitHub en Azure AD B2C

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** B2C: administración de identidades de consumidor B2B/B2C

Ahora, puede agregar Twitter o GitHub como proveedores de identidades en Azure AD B2C. Twitter está pasando de la versión preliminar pública a la versión de disponibilidad general. GitHub se va a publicar en la versión preliminar pública.

Para más información, consulte [¿Qué es la colaboración B2B de Azure AD?](../external-identities/what-is-b2b.md)

---

### <a name="restrict-browser-access-using-intune-managed-browser-with-azure-ad-application-based-conditional-access-for-ios-and-android"></a>Restricción del acceso de explorador mediante Intune Managed Browser con acceso condicional basado en aplicaciones de Azure AD para iOS y Android

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

**Ahora en versión preliminar pública**

**SSO de Intune Managed Browser:** los empleados pueden utilizar el inicio de sesión único por los clientes nativos (por ejemplo, Microsoft Outlook) e Intune Managed Browser para todas las aplicaciones de Azure AD conectados.

**Compatibilidad con el acceso condicional de Intune Managed Browser:** ahora puede requerir que los empleados usen Intune Managed Browser mediante directivas de acceso condicional basado en aplicaciones.

Lea más sobre esto en nuestra [entrada de blog](https://cloudblogs.microsoft.com/enterprisemobility/2018/03/15/the-intune-managed-browser-now-supports-azure-ad-sso-and-conditional-access/).

Para más información, consulte:

- [Configuración del acceso condicional basado en aplicaciones](../conditional-access/app-based-conditional-access.md)

- [Configuración de directivas de Managed Browser](/mem/intune/apps/manage-microsoft-edge)

---

### <a name="app-proxy-cmdlets-in-powershell-ga-module"></a>Cmdlets del proxy de aplicación en el módulo de disponibilidad general de PowerShell

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Control de acceso

Ahora, los cmdlets del proxy de aplicación pueden utilizarse en el módulo de disponibilidad general de PowerShell. Es necesario mantener actualizados los módulos de PowerShell; si lleva más de un año de retraso, es posible que algunos cmdlets dejen de funcionar.

Para más información, consulte [AzureAD](/powershell/module/Azuread/).

---

### <a name="office-365-native-clients-are-supported-by-seamless-sso-using-a-non-interactive-protocol"></a>Loa clientes nativos de Office 365 son compatibles con SSO de conexión directa utilizando un protocolo no interactivo

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Los usuarios que utilizan clientes nativos de Office 365 (versión 16.0.8730.xxxx y posteriores) pueden iniciar sesión sin que sea necesaria su intervención mediante SSO de conexión directa. Esta funcionalidad es posible gracias a la incorporación de un protocolo no interactivo (WS-Trust) en Azure AD.

Para más información, consulte [¿Cómo funciona el inicio de sesión en un cliente nativo con SSO de conexión directa?](../hybrid/how-to-connect-sso-how-it-works.md#how-does-sign-in-on-a-native-client-with-seamless-sso-work)

---

### <a name="users-get-a-silent-sign-on-experience-with-seamless-sso-if-an-application-sends-sign-in-requests-to-azure-ads-tenant-endpoints"></a>Con SSO de conexión directa, no es necesaria la intervención del usuario para iniciar sesión cuando una aplicación envía solicitudes de inicio de sesión a puntos de conexión de inquilinos de Azure AD.

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Con el inicio de sesión único de conexión directa, los usuarios no necesitan intervenir para iniciar sesión si una aplicación (por ejemplo, `https://contoso.sharepoint.com`) envía solicitudes de inicio de sesión a puntos de conexión de inquilinos de Azure AD (por ejemplo, `https://login.microsoftonline.com/contoso.com/<..>` o `https://login.microsoftonline.com/<tenant_ID>/<..>`), en lugar de a puntos de conexión comunes de Azure AD (`https://login.microsoftonline.com/common/<...>`).

Para más información, consulte [Inicio de sesión único de conexión directa de Azure Active Directory](../hybrid/how-to-connect-sso.md).

---

### <a name="need-to-add-only-one-azure-ad-url-instead-of-two-urls-previously-to-users-intranet-zone-settings-to-roll-out-seamless-sso"></a>Solo es necesario agregar una dirección URL de Azure AD, en lugar de las dos que se requerían anteriormente, en la configuración de la zona de intranet de los usuarios para activar el SSO de conexión directa

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Si desea activar el SSO de conexión directa para los usuarios, solo debe agregar una dirección URL de Azure AD en la configuración de la zona de intranet de los usuarios utilizando una directiva de grupo de Active Directory: `https://autologon.microsoftazuread-sso.com`. Anteriormente, era necesario que los clientes agregaran dos direcciones URL.

Para más información, consulte [Inicio de sesión único de conexión directa de Azure Active Directory](../hybrid/how-to-connect-sso.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Nuevas aplicaciones federadas disponibles en la Galería de aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En abril de 2018, hemos agregado estas 15 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[Boxcryptor](../saas-apps/boxcryptor-tutorial.md), [CylancePROTECT](../saas-apps/cylanceprotect-tutorial.md), Wrike, [SignalFx](../saas-apps/signalfx-tutorial.md), Assistant by FirstAgenda, [YardiOne](../saas-apps/yardione-tutorial.md), Vtiger CRM, inwink, [Amplitude](../saas-apps/amplitude-tutorial.md), [Spacio](../saas-apps/spacio-tutorial.md), [ContractWorks](../saas-apps/contractworks-tutorial.md), [Bersin](../saas-apps/bersin-tutorial.md), [Mercell](../saas-apps/mercell-tutorial.md), [Trisotech Digital Enterprise Server](../saas-apps/trisotechdigitalenterpriseserver-tutorial.md) y [Qumu Cloud](../saas-apps/qumucloud-tutorial.md).

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md).

Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="pim-for-azure-resources-is-generally-available"></a>PIM de Azure Resources está disponible con carácter general

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Si utiliza Azure AD Privileged Identity Management para los roles de directorio, ahora puede utilizar las funcionalidades de asignación y acceso con límite de tiempo de PIM con los roles de Recursos de Azure, como Suscripciones, Grupos de recursos, Máquinas virtuales y cualquier otro recurso compatible con Azure Resource Manager. Puede forzar la aplicación de la autenticación multifactor al activar roles Just-In-Time y programar activaciones de forma coordinada con los periodos de cambio aprobados. Además, esta versión incorpora mejoras que no está disponibles en la versión preliminar pública, como una interfaz de usuario actualizada, flujos de trabajo de aprobación y la capacidad de ampliar roles que van a expirar en breve y de renovar roles que han expirado.

Para obtener más información, consulte [PIM para recursos de Azure (versión preliminar)](../privileged-identity-management/azure-pim-resource-rbac.md).

---

### <a name="adding-optional-claims-to-your-apps-tokens-public-preview"></a>Incorporación de notificaciones opcionales en los tokens de aplicaciones (versión preliminar pública)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Ahora, las aplicaciones de Azure AD pueden solicitar notificaciones personalizadas u opcionales en los tokens SAML o JWT.  Estas notificaciones sobre el usuario o el inquilino no se incluyen de forma predeterminada en el token por restricciones de tamaño o aplicabilidad.  Actualmente, esta funcionalidad está en versión preliminar pública para las aplicaciones de Azure AD de los puntos de conexión 1.0 y 2.0.  Consulte la documentación para obtener información acerca de qué notificaciones pueden agregarse y cómo editar el manifiesto de aplicación para solicitarlas.

Para más información, consulte [Optional claims in Azure AD](../develop/active-directory-optional-claims.md) (Notificaciones opcionales de Azure AD)///.

---

### <a name="azure-ad-supports-pkce-for-more-secure-oauth-flows"></a>Azure AD permite usar PKCE para disponer de flujos de OAuth más seguros

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Los documentos de Azure AD se han actualizado para que tengan en cuenta la compatibilidad con PKCE, lo que permite una comunicación más segura durante el flujo de concesión del código de autorización de OAuth 2.0.  Los puntos de conexión de la versión 1.0 y 2.0 admiten S256 y code_challenges de texto no cifrado.

Para más información, consulte [Solicitud de un código de autorización](../develop/v2-oauth2-auth-code-flow.md#request-an-authorization-code).

---

### <a name="support-for-provisioning-all-user-attribute-values-available-in-the-workday-get_workers-api"></a>Compatibilidad con el aprovisionamiento de todos los valores de atributo de usuario disponibles en Workday Get_Workers API

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aprovisionamiento de aplicaciones Integración de terceros

Ahora, la versión preliminar pública de aprovisionamiento de entrada de Workday a Active Directory y Azure AD permite extraer y aprovisionar todos los valores de atributo disponibles en Workday Get_Workers API. De este modo, se amplía la compatibilidad con cientos de atributos estándar y personalizados, que se suman a los que ya venían con la versión inicial del conector de aprovisionamiento de entrada de Workday.

Para más información, consulte: [Personalización de la lista de atributos de usuario de Workday](../saas-apps/workday-inbound-tutorial.md#customizing-the-list-of-workday-user-attributes)

---

### <a name="changing-group-membership-from-dynamic-to-static-and-vice-versa"></a>Cambio de la pertenencia a grupos de dinámica a estática, y viceversa

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de grupos Colaboración

Es posible cambiar cómo se administra la pertenencia a un grupo. Esto es útil cuando desea mantener el mismo nombre de grupo y el identificador en el sistema, por lo que cualquier referencia existente al grupo sigue siendo válida; crear un nuevo grupo requeriría actualizar esas referencias.
Hemos actualizado el centro de administración de Azure AD para incorporar la compatibilidad con esta funcionalidad. Ahora, los clientes pueden cambiar los grupos existentes para que tengan una pertenencia dinámica en lugar de una pertenencia asignada, y viceversa. Los cmdlets de PowerShell existentes seguirán estando disponibles.

Para obtener más información, consulte [Reglas de pertenencia dinámica a grupos de Azure Active Directory](../enterprise-users/groups-dynamic-membership.md).

---

### <a name="improved-sign-out-behavior-with-seamless-sso"></a>Comportamiento de cierre de sesión mejorado con SSO de conexión directa

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Anteriormente, aunque los usuarios cerraran sesión explícitamente en una aplicación protegida con Azure AD, la sesión se iniciaba de nuevo automáticamente mediante SSO de conexión directa si intentaban volver a acceder a una aplicación de Azure AD en la red corporativa desde los dispositivos unidos al dominio. Con este cambio, se puede cerrar sesión.  De este modo, los usuarios pueden elegir otra cuenta de Azure AD al iniciar sesión de nuevo, en lugar de la que la sesión se inicie automáticamente con el SSO de conexión directa.

Para más información, consulte [Inicio de sesión único de conexión directa de Azure Active Directory](../hybrid/how-to-connect-sso.md).

---

### <a name="application-proxy-connector-version-154020-released"></a>Se ha publicado la versión 1.5.402.0 del conector del proxy de aplicación

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Proxy de aplicaciones Seguridad y protección de la identidad

Esta versión del conector se irá implementando gradualmente hasta noviembre. Esta nueva versión del conector incluye los siguientes cambios:

- Ahora, el conector establece cookies de nivel de dominio en lugar de cookies de nivel de subdominio. De este modo, la experiencia de SSO resulta más fluida y se evita el envío de peticiones de autenticación redundantes.
- Se admiten solicitudes de codificación fragmentada.
- Se ha mejorado la supervisión del mantenimiento del conector.
- Se han solucionado algunos errores y se han hecho mejoras en la estabilidad.

Para más información, consulte [Descripción de los conectores de Azure Active Directory Application Proxy](../manage-apps/application-proxy-connectors.md).

---

## <a name="february-2018"></a>Febrero de 2018

### <a name="improved-navigation-for-managing-users-and-groups"></a>Navegación mejorada para administrar usuarios y grupos

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Administración de directorios Directorio

Se ha simplificado la experiencia de navegación para administrar usuarios y grupos. Ahora puede ir directamente desde la información general del directorio a la lista de todos los usuarios, con un acceso más fácil a la lista de usuarios eliminados. Ahora puede ir directamente desde la información general del directorio a la lista de todos los grupos, con un acceso más fácil a la configuración de administración de grupos. Y también en la página de información general del directorio puede buscar un usuario, grupo, aplicación empresarial o registro de la aplicación.

---

### <a name="availability-of-sign-ins-and-audit-reports-in-microsoft-azure-operated-by-21vianet-azure-china-21vianet"></a>Disponibilidad de los informes de inicios de sesión y auditoría en Microsoft Azure controlado por 21Vianet (Azure China 21Vianet)

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure Stack Supervisión e informes

Los informes de registro de actividad de Azure AD ya están disponibles en Microsoft Azure controlado por instancias de 21Vianet (Azure China 21Vianet). Se incluyen los siguientes registros:

- **Registros de actividad de inicios de sesión**: incluye todos los registros de inicios de sesión asociados con el inquilino.

- **Registros de auditoría de contraseñas de autoservicio**: incluye todos los registros de auditoría de SSPR.

- **Registros de auditoría de administración de directorios**: incluye todos los registros de auditoría relacionados con la administración de directorios como, por ejemplo, administración de usuarios, administración de aplicaciones y otros.

Con estos registros, puede obtener información detallada sobre el funcionamiento de su entorno. Los datos proporcionados le permiten:

- Determinar cómo utilizan los usuarios las aplicaciones y servicios.

- Solucionar problemas que impiden a los usuarios finalizar su trabajo.

Para más información sobre cómo usar estos informes, consulte [Informes de Azure Active Directory](../reports-monitoring/overview-reports.md).

---

### <a name="use-report-reader-role-non-admin-role-to-view-azure-ad-activity-reports"></a>Use el rol "Lector de informes" (rol que no es de administrador) para ver los informes de actividad de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Informes Supervisión e informes

Como parte de los comentarios de los clientes para habilitar los roles que no son de administrador para acceder a los registros de actividades de Azure AD, hemos habilitado la posibilidad de que los usuarios con el rol "Lector de informes" puedan acceder a las actividades de inicios de sesión y de auditoría en Azure Portal, así como mediante Microsoft Graph API.

Para más información sobre cómo usar estos informes, consulte [Informes de Azure Active Directory](../reports-monitoring/overview-reports.md).

---

### <a name="employeeid-claim-available-as-user-attribute-and-user-identifier"></a>Notificación de EmployeeID disponible como atributo de usuario e identificador de usuario

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales SSO

Puede configurar **EmployeeID** como el identificador de usuario y el atributo de usuario para los usuarios miembros y para los invitados B2B en aplicaciones de inicio de sesión basadas en SAML desde la interfaz de usuario de la aplicación empresarial.

Para más información, consulte [Personalización de las notificaciones emitidas en el token SAML para aplicaciones empresariales en Azure Active Directory](../develop/active-directory-saml-claims-customization.md).

---

### <a name="simplified-application-management-using-wildcards-in-azure-ad-application-proxy"></a>Administración simplificada de aplicaciones mediante caracteres comodín en Azure AD Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Autenticación de usuarios

Para facilitar la implementación de aplicaciones y reducir la carga administrativa, ahora se admite la posibilidad de publicar aplicaciones mediante caracteres comodín. Para publicar una aplicación con comodín, puede seguir el flujo de publicación de aplicaciones estándar, pero usando un carácter comodín en las direcciones URL internas y externas.

Para más información, consulte [Aplicaciones con comodín en Azure Active Directory Application Proxy](../manage-apps/application-proxy-wildcard.md).

---

### <a name="new-cmdlets-to-support-configuration-of-application-proxy"></a>Nuevos cmdlets para admitir la configuración de Application Proxy

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Plataforma

La versión preliminar más reciente del módulo Azure AD PowerShell contiene nuevos cmdlets que permiten a los clientes configurar aplicaciones de Application Proxy mediante PowerShell.

Los nuevos cmdlets son:

- Get-AzureADApplicationProxyApplication
- Get-AzureADApplicationProxyApplicationConnectorGroup
- Get-AzureADApplicationProxyConnector
- Get-AzureADApplicationProxyConnectorGroup
- Get-AzureADApplicationProxyConnectorGroupMembers
- Get-AzureADApplicationProxyConnectorMemberOf
- New-AzureADApplicationProxyApplication
- New-AzureADApplicationProxyConnectorGroup
- Remove-AzureADApplicationProxyApplication
- Remove-AzureADApplicationProxyApplicationConnectorGroup
- Remove-AzureADApplicationProxyConnectorGroup
- Set-AzureADApplicationProxyApplication
- Set-AzureADApplicationProxyApplicationConnectorGroup
- Set-AzureADApplicationProxyApplicationCustomDomainCertificate
- Set-AzureADApplicationProxyApplicationSingleSignOn
- Set-AzureADApplicationProxyConnector
- Set-AzureADApplicationProxyConnectorGroup

---

### <a name="new-cmdlets-to-support-configuration-of-groups"></a>Nuevos cmdlets para admitir la configuración de grupos

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Proxy de aplicaciones Plataforma

La versión más reciente del módulo de Azure AD PowerShell contiene cmdlets para administrar grupos en Azure AD. Estos cmdlets no estaban disponibles anteriormente en el módulo AzureADPreview y ahora se han agregado al módulo AzureAD.

Los cmdlets de grupo que ya se han publicado con disponibilidad general son:

- Get-AzureADMSGroup
- New-AzureADMSGroup
- Remove-AzureADMSGroup
- Set-AzureADMSGroup
- Get-AzureADMSGroupLifecyclePolicy
- New-AzureADMSGroupLifecyclePolicy
- Remove-AzureADMSGroupLifecyclePolicy
- Add-AzureADMSLifecyclePolicyGroup
- Remove-AzureADMSLifecyclePolicyGroup
- Reset-AzureADMSLifeCycleGroup
- Get-AzureADMSLifecyclePolicyGroup

---

### <a name="a-new-release-of-azure-ad-connect-is-available"></a>Está disponible una nueva versión de Azure AD Connect

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Sincronización de AD Plataforma

Azure AD Connect es la herramienta preferida para sincronizar datos entre Azure AD y los orígenes de datos locales, incluidos Windows Server Active Directory y LDAP.

>[!Important]
>Esta compilación introduce cambios en las reglas de sincronización y en el esquema. El servicio de sincronización de Azure AD Connect desencadenará pasos de importación completa y sincronización completa después de una actualización. Para obtener información sobre cómo cambiar este comportamiento, consulte [Aplazamiento de la sincronización completa después de la actualización](../hybrid/how-to-upgrade-previous-version.md#how-to-defer-full-synchronization-after-upgrade).

Esta versión incluye las siguientes actualizaciones y cambios:

**Problemas corregidos:**

- Corrección de la ventana de sincronización en las tareas en segundo plano para la página de filtrado de particiones al cambiar a la página siguiente.

- Se ha corregido un error que provocó una infracción de acceso durante la acción personalizada ConfigDB.

- Se ha corregido un error para recuperarse del tiempo de espera de la conexión de SQL.

- Se ha corregido un error que provocaba que los certificados con caracteres comodín de SAN generaran un error al realizar una comprobación de requisitos previos.

- Se ha corregido un error que provocaba que miiserver.exe se bloqueara durante una exportación del conector de Azure AD.

- Se ha corregido un error en el que un intento de contraseña incorrecta registrado en el controlador de dominio al ejecutarse provocaba que el asistente de Azure AD Connect cambiara la configuración.

**Nuevas características y mejoras**

- Telemetría de aplicaciones: los administradores pueden activar o desactivar este tipo de datos.

- Datos de mantenimiento de Azure AD: los administradores deben visitar el portal de mantenimiento para controlar la configuración de mantenimiento. Una vez se haya cambiado la directiva del servicio, los agentes la leerán y la aplicarán.

- Se han agregado acciones de configuración de escritura diferida de dispositivo y una barra de progreso para la inicialización de la página.

- Se han mejorado los diagnósticos generales con un informe HTML y una recopilación completa de datos en un informe HTML o de texto ZIP.

- Se ha mejorado la confiabilidad de la actualización automática y se ha agregado telemetría adicional para asegurarse de que se puede determinar el mantenimiento del servidor.

- Se han restringido los permisos disponibles para las cuentas con privilegios de la cuenta del conector AD. Para las nuevas instalaciones, el asistente restringe los permisos que las cuentas con privilegios tienen en la cuenta de MSOL tras la creación de esa cuenta. Los cambios afectan a las instalaciones rápidas y a las instalaciones personalizadas con la opción de creación automática de cuenta.

- Se ha cambiado el instalador, por lo que no se requiere el privilegio de asociación de seguridad en una instalación limpia de AADConnect.

- Se ha agregado una utilidad nueva para solucionar problemas de sincronización de un objeto específico. Actualmente, la utilidad comprueba los siguientes aspectos:

    - Error de coincidencia de UserPrincipalName entre el objeto de usuario sincronizado y la cuenta de usuario del inquilino de Azure AD.

    - Si se filtra el objeto de sincronización debido al filtrado de dominio

    - Si se filtra el objeto de sincronización debido al filtrado de unidad organizativa (UO)

- Se ha agregado una utilidad nueva para sincronizar el hash de contraseña actual almacenado en el Active Directory local para una cuenta de usuario específica. La utilidad no requiere un cambio de contraseña.

---

### <a name="applications-supporting-intune-app-protection-policies-added-for-use-with-azure-ad-application-based-conditional-access"></a>Se han agregado aplicaciones compatibles con directivas de Intune App Protection para usarlas con el acceso condicional basado en aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Acceso condicional Seguridad y protección de la identidad

Se han agregado más aplicaciones que admiten el acceso condicional basado en aplicaciones. Ahora, ya puede acceder a Office 365 y otras aplicaciones en la nube conectadas a Azure AD mediante estas aplicaciones cliente aprobadas.

Las siguientes aplicaciones se agregarán a finales del mes de febrero:

- Microsoft Power BI

- Microsoft Launcher

- Microsoft Invoicing

Para más información, consulte:

- [Requisito de aplicación cliente aprobada](../conditional-access/concept-conditional-access-conditions.md#client-apps)
- [Acceso condicional basado en aplicaciones de Azure AD](../conditional-access/app-based-conditional-access.md)

---

### <a name="terms-of-use-update-to-mobile-experience"></a>Actualización de los términos de uso relacionados con la experiencia móvil

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Cuando aparecen los términos de uso, puede hacer clic en **¿Tiene problemas con la visualización? Haga clic aquí.** Si hace clic en este vínculo se abren los términos de uso de forma nativa en el dispositivo. Independientemente del tamaño de fuente en el documento o el tamaño de pantalla del dispositivo, puede acercar o alejar y leer el documento según sea necesario.

---

## <a name="january-2018"></a>Enero de 2018

### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Nuevas aplicaciones federadas disponibles en la Galería de aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En enero de 2018, se agregaron a la galería de aplicaciones las siguientes aplicaciones nuevas compatibles con la federación:

[IBM OpenPages](../saas-apps/ibmopenpages-tutorial.md), [OneTrust Privacy Management Software](../saas-apps/onetrust-tutorial.md), [Dealpath](../saas-apps/dealpath-tutorial.md), [IriusRisk Federated Directory y [Fidelity NetBenefits](../saas-apps/fidelitynetbenefits-tutorial.md).

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md).

Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="sign-in-with-additional-risk-detected"></a>Inicio de sesión con riesgo adicional detectado

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Identity Protection Seguridad y protección de la identidad

La perspectiva que se obtiene de una detección de riesgos identificada está asociada a su suscripción de Azure AD. Con la edición de Azure AD Premium P2, obtiene la información más detallada acerca de todas las detecciones subyacentes.

Con la edición de Azure AD Premium P1, las detecciones que no están cubiertas por su licencia aparecen como la detección de riesgos Inicio de sesión con riesgo adicional detectado.

Para más información, consulte [Detecciones de riesgos de Azure Active Directory](../identity-protection/overview-identity-protection.md).

---

### <a name="hide-office-365-applications-from-end-users-access-panels"></a>Ocultación de aplicaciones de Office 365 de los paneles de acceso del usuario final

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones SSO

Ahora puede administrar mejor el modo en que se muestran las aplicaciones de Office 365 en los paneles de acceso de sus usuarios mediante una nueva configuración de usuario. Esta opción le resultará útil para reducir el número de aplicaciones de los paneles de acceso de los usuarios si prefiere mostrar solo las aplicaciones de Office en el Portal de Office. La configuración se encuentra en **Configuración de usuario** y tiene la etiqueta **Los usuarios solo pueden ver las aplicaciones de Office 365 en el Portal de Office 365**.

Para obtener más información, consulte [Ocultación de una aplicación de la experiencia del usuario en Azure Active Directory](../manage-apps/hide-application-from-user-portal.md).

---

### <a name="seamless-sign-into-apps-enabled-for-password-sso-directly-from-apps-url"></a>Inicio de sesión sin problemas en aplicaciones habilitadas para SSO de contraseña directamente desde la dirección URL de la aplicación

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones SSO

La extensión del explorador Mis aplicaciones ahora está disponible en una cómoda herramienta que ofrece la funcionalidad de inicio de sesión único Mis aplicaciones como acceso directo del explorador. Tras la instalación, los usuarios verán un icono de gofre en el explorador que proporciona acceso rápido a las aplicaciones. Ahora, los usuarios podrán disfrutar de las ventajas siguientes:

- Capacidad para iniciar sesión directamente en aplicaciones de SSO de contraseña desde la página de inicio de sesión de la aplicación.
- Inicio de cualquier aplicación mediante la característica de búsqueda rápida.
- Accesos directos a aplicaciones usadas recientemente desde la extensión.
- La extensión está disponible en Microsoft Edge, Chrome y Firefox.

Para obtener más información, consulte [Extensión de inicio de sesión seguro de Mis aplicaciones](../user-help/my-apps-portal-end-user-access.md#download-and-install-the-my-apps-secure-sign-in-extension).

---

### <a name="azure-ad-administration-experience-in-azure-classic-portal-has-been-retired"></a>Retirada de la experiencia de administración de Azure AD del Portal de Azure clásico

**Tipo:** **Categoría del servicio:** En desuso **Funcionalidad del producto:** Azure AD Directorio

El 8 de enero de 2018, la experiencia de administración de Azure AD se retiró del Portal de Azure clásico. Esto tuvo lugar junto con la retirada del Portal de Azure clásico. En el futuro, deberá usar el [Centro de administración de Azure AD](https://aad.portal.azure.com) para todas las tareas de administración de Azure AD basadas en el portal.

---

### <a name="the-phonefactor-web-portal-has-been-retired"></a>El portal web PhoneFactor se ha retirado.

**Tipo:** **Categoría del servicio:** En desuso **Funcionalidad del producto:** Azure AD Directorio

El 8 de enero de 2018 se retiró el portal web PhoneFactor. Este portal se usaba para la administración del servidor MFA; sin embargo, sus funciones se han movido a Azure Portal en portal.azure.com.

La configuración de MFA se encuentra en: **Azure Active Directory \> Servidor MFA**

---

### <a name="deprecate-azure-ad-reports"></a>Informes de Azure AD obsoletos

**Tipo:** **Categoría del servicio:** En desuso **Funcionalidad del producto**: Informes Administración del ciclo de vida de la identidad


Con la disponibilidad general de la nueva consola de administración de Azure Active Directory y las nuevas API disponibles para los informes de actividad y seguridad, las API de informes que se encontraban en el punto de conexión "/reports" se retiraron el 31 de diciembre de 2017.

**¿Qué está disponible?**

Como parte de la transición a la nueva consola de administración, existen dos nuevas API disponibles para recuperar los registros de actividad de Azure AD. El nuevo conjunto de API proporciona más funciones de filtro y ordenación, así como actividades de auditoría e inicio de sesión. Los datos que estaban disponibles anteriormente en los informes de seguridad ahora son accesibles a través de la API de detecciones de riesgos de Identity Protection en Microsoft Graph.

Para más información, consulte:

- [Introducción a la API de informes de Azure Active Directory](../reports-monitoring/concept-reporting-api.md)

- [Introducción a Azure Active Directory Identity Protection y Microsoft Graph](../identity-protection/howto-identity-protection-graph-api.md)

---

## <a name="december-2017"></a>Diciembre de 2017

### <a name="terms-of-use-in-the-access-panel"></a>Términos de uso del panel de acceso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Ahora puede ir al panel de acceso y ver los términos de uso previamente aceptados.

Siga estos pasos:

1. Abra el [portal de Mis aplicaciones](https://myapps.microsoft.com) e inicie sesión.

2. En la esquina superior derecha, seleccione su nombre y seleccione **Perfil** en la lista.

3. En su **Perfil**, seleccione **Revisar los términos de uso**.

4. Ahora podrá revisar los términos de uso que aceptó.

Para obtener más información, consulte [Característica Términos de uso de Azure Active Directory (versión preliminar)](../conditional-access/terms-of-use.md).

---

### <a name="new-azure-ad-sign-in-experience"></a>Nueva experiencia de inicio de sesión de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Autenticación de usuarios

Azure AD y las interfaces de usuario del sistema de identidades de cuentas de Microsoft se han rediseñado para ofrecer un aspecto más homogéneo. Además, la página de inicio de sesión de Azure AD primero recopila el nombre de usuario, seguido de la credencial en una segunda pantalla.

Para obtener más información, consulte [The new Azure AD Signin Experience is now in Public Preview](https://cloudblogs.microsoft.com/enterprisemobility/2017/08/02/the-new-azure-ad-signin-experience-is-now-in-public-preview/) (Nueva experiencia de inicio de sesión de Azure AD disponible en versión preliminar pública).

---

### <a name="fewer-sign-in-prompts-a-new-keep-me-signed-in-experience-for-azure-ad-sign-in"></a>Menos solicitudes de inicio de sesión: nueva experiencia "Mantener la sesión iniciada" para el inicio de sesión de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Autenticación de usuarios

Hemos sustituido la casilla **Mantener la sesión iniciada** de la página de inicio de sesión de Azure AD por un nuevo mensaje que se muestra al realizar la autenticación correctamente.

Si se responde **Sí** a este mensaje, el servicio proporciona un token de actualización persistente. Este es el mismo comportamiento que se produce cuando se activa la casilla **Mantener la sesión iniciada** en la experiencia antigua. Para los inquilinos federados, este mensaje se muestra al autenticarse correctamente en el servicio federado.

Para más información, consulte [Fewer sign-in prompts: The new "keep me signed in" experience for Azure AD is in preview](https://cloudblogs.microsoft.com/enterprisemobility/2017/09/19/fewer-login-prompts-the-new-keep-me-signed-in-experience-for-azure-ad-is-in-preview/) (Menos avisos de inicio de sesión: la nueva experiencia "Mantener la sesión iniciada" para Azure AD está en versión preliminar).

---

### <a name="add-configuration-to-require-the-terms-of-use-to-be-expanded-prior-to-accepting"></a>Agregar configuración para exigir expandir los términos de uso antes de su aceptación

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Se ha agregado una opción para los administradores con el fin de exigir a usuarios finales que expandan los términos de uso antes de aceptarlos.

Seleccione **Activado** o **Desactivado** para exigir que los usuarios expandan los términos de uso. El valor **Activado** exige a usuarios que lean los términos de uso antes de aceptarlos.

Para obtener más información, consulte [Característica Términos de uso de Azure Active Directory (versión preliminar)](../conditional-access/terms-of-use.md).

---

### <a name="scoped-activation-for-eligible-role-assignments"></a>Activación de ámbito para las asignaciones de roles válidos

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Puede usar la activación con ámbito para activar las asignaciones de roles de recursos de Azure válidas con menos autonomía que los valores predeterminados de las asignaciones originales. Un ejemplo de esto es si se le asigna como propietario de una suscripción en su inquilino. Con la activación con ámbito, podrá activar el rol de propietario para hasta cinco de los recursos incluidos en la suscripción (por ejemplo, los grupos de recursos y las máquinas virtuales). La definición del ámbito de la activación puede reducir la posibilidad de ejecutar cambios no deseados en recursos importantes de Azure.

Para más información, vea [¿Qué es Azure AD Privileged Identity Management?](../privileged-identity-management/pim-configure.md).

---

### <a name="new-federated-apps-in-the-azure-ad-app-gallery"></a>Nuevas aplicaciones federadas en la galería de aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Aplicaciones empresariales Integración de terceros

En diciembre de 2017, hemos agregado estas 15 nuevas aplicaciones con compatibilidad con la federación a nuestra galería de aplicaciones:

[Accredible](../saas-apps/accredible-tutorial.md), Adobe Experience Manager, [EFI Digital StoreFront](../saas-apps/efidigitalstorefront-tutorial.md), [Communifire](../saas-apps/communifire-tutorial.md) CybSafe, [FactSet](../saas-apps/factset-tutorial.md), [IMAGE WORKS](../saas-apps/imageworks-tutorial.md), [MOBI](../saas-apps/mobi-tutorial.md), [Integración de Azure AD con MobileIron](../saas-apps/mobileiron-tutorial.md), [Reflektive](../saas-apps/reflektive-tutorial.md), [SAML SSO for Bamboo by resolution GmbH](../saas-apps/bamboo-tutorial.md), [SAML SSO for Bitbucket by resolution GmbH](../saas-apps/bitbucket-tutorial.md), [Vodeclic](../saas-apps/vodeclic-tutorial.md), WebHR, Zenegy Azure AD Integration.

Para obtener más información acerca de las aplicaciones, consulte [Integración de aplicación SaaS con Azure Active Directory](../saas-apps/tutorial-list.md).

Para obtener más información para que una aplicación se muestre en la galería de aplicaciones de Azure AD, consulte [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="approval-workflows-for-azure-ad-directory-roles"></a>Flujos de trabajo de aprobación de roles de directorio de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

El flujo de trabajo de aprobación de roles de directorio de Azure AD está disponible de manera general.

Con el flujo de trabajo de aprobación, los administradores de roles con privilegios pueden exigir que los miembros de roles aptos soliciten la activación de roles para poder usar el rol con privilegios. Ahora es posible delegar responsabilidades de aprobación en múltiples usuarios y grupos. Los miembros del rol aptos recibirán notificaciones cuando termine la aprobación y sus roles estén activos.

---

### <a name="pass-through-authentication-skype-for-business-support"></a>Autenticación de paso a través: Compatibilidad con Skype Empresarial

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Autenticaciones (inicios de sesión) Autenticación de usuarios

Ahora, la autenticación de paso a través admite inicios de sesión de usuarios en aplicaciones cliente de Skype Empresarial compatibles con la autenticación moderna, incluidas las topologías híbridas y en línea.

Para obtener más información, consulte [Topologías de Skype Empresarial compatibles con la autenticación moderna](/skypeforbusiness/plan-your-deployment/modern-authentication/topologies-supported).

---

### <a name="updates-to-azure-ad-privileged-identity-management-for-azure-rbac-preview"></a>Actualizaciones de Azure AD Privileged Identity Management para RBAC de Azure (versión preliminar)

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Con la actualización de versión preliminar pública de Azure AD Privileged Identity Management (PIM) para el control de acceso basado en rol (RBAC) de Azure, ahora podrá realizar lo siguiente:

* Usar Just Enough Administration.
* Solicitar aprobación para activar roles de recursos.
* Programar la activación futura de un rol que requiere la aprobación de roles de Azure AD y de Azure.

Para obtener más información, consulte [PIM para recursos de Azure (versión preliminar)](../privileged-identity-management/azure-pim-resource-rbac.md).

---

## <a name="november-2017"></a>Noviembre de 2017

### <a name="access-control-service-retirement"></a>Retirada del servicio de control de acceso

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto**: Access Control Service Servicio Access Control

Azure Active Directory Access Control (también conocido como Access Control Service) se retirará a finales de 2018. En las próximas semanas se ofrecerá más información, como, por ejemplo, una programación detallada y una guía de migración de alto nivel. Puede dejar comentarios en esta página con preguntas acerca de Access Control Service y un miembro del equipo le responderá.

---

### <a name="restrict-browser-access-to-the-intune-managed-browser"></a>Restringir el acceso desde el explorador a Intune Managed Browser

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

El uso de Intune Managed Browser como aplicación aprobada le permite restringir el acceso desde el explorador a Office 365 y otras aplicaciones en la nube conectadas a Azure AD.

Ahora podrá configurar la siguiente condición para el acceso condicional basado en aplicaciones:

**Aplicaciones cliente:** Browser

**¿Cuál es el efecto del cambio?**

En la actualidad, el acceso está bloqueado cuando se usa esta condición. Cuando la vista previa está disponible, todos los accesos requerirán el uso de la aplicación Manager Browser.

Busque esta funcionalidad y más información en las próximas entradas de blogs y notas de versión.

Para obtener más información, consulte [Acceso condicional en Azure AD](../conditional-access/overview.md).

---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>Nuevas aplicaciones cliente aprobadas para el acceso condicional basado en aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

Las siguientes aplicaciones se incluyen en la lista de [aplicaciones cliente aprobadas](../conditional-access/concept-conditional-access-conditions.md#client-apps):

- [Microsoft Kaizala](https://www.microsoft.com/garage/profiles/kaizala/)
- Microsoft StaffHub

Para más información, consulte:

- [Requisito de aplicación cliente aprobada](../conditional-access/concept-conditional-access-conditions.md#client-apps)
- [Acceso condicional basado en aplicaciones de Azure AD](../conditional-access/app-based-conditional-access.md)

---

### <a name="terms-of-use-support-for-multiple-languages"></a>Disponibilidad de los términos de uso en varios idiomas

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Ahora, los administradores pueden crear nuevos términos de uso que contengan varios documentos PDF. Puede etiquetar estos documentos PDF con el idioma correspondiente. De este modo, se mostrará a los usuarios el documento PDF en el idioma correspondiente a sus preferencias. Si no hay ninguna coincidencia, se muestra el idioma predeterminado.

---

### <a name="real-time-password-writeback-client-status"></a>Estado del cliente de escritura diferida de contraseñas en tiempo real

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Autoservicio de restablecimiento de contraseña Autenticación de usuarios

Ahora podrá revisar el estado de su cliente de escritura diferida de contraseñas local. Esta opción está disponible en la sección **Integración local** de la página [Restablecimiento de contraseña](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/PasswordReset).

Si hay problemas con la conexión al cliente de escritura diferida local, se mostrará un mensaje de error con los elementos siguientes:

- Información del motivo por el que no puede conectarse a su cliente de escritura diferida local.
- Un vínculo a documentación que le ayuda a resolver el problema.

Para obtener más información, consulte [Integración local](../authentication/concept-sspr-howitworks.md#on-premises-integration).

---

### <a name="azure-ad-app-based-conditional-access"></a>Acceso condicional basado en aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Azure AD Protección y seguridad de la identidad

Ahora puede restringir el acceso a Office 365 y a otras aplicaciones en la nube conectadas a Azure AD para [aplicaciones cliente aprobadas](../conditional-access/concept-conditional-access-conditions.md#client-apps) que admiten directivas de Intune App Protection mediante el [acceso condicional basado en aplicaciones de Azure AD](../conditional-access/app-based-conditional-access.md). Las directivas de Intune App Protection se utilizan para configurar y proteger los datos de empresa en estas aplicaciones cliente.

Al combinar directivas de acceso condicional [basado en aplicaciones](../conditional-access/app-based-conditional-access.md) con las directivas de acceso condicional [basado en dispositivos](../conditional-access/require-managed-devices.md), tiene la flexibilidad necesaria para proteger los datos de dispositivos personales y de la empresa.

Ahora puede usar los siguientes controles y condiciones con el acceso condicional basado en aplicaciones:

**Condición de plataforma admitida**

- iOS
- Android

**Condición de aplicaciones cliente**

- Aplicaciones móviles y aplicaciones de escritorio

**Control de acceso**

- Requerir aplicación cliente aprobada

Para obtener más información, consulte [Acceso condicional basado en aplicaciones de Azure AD](../conditional-access/app-based-conditional-access.md).

---

### <a name="manage-azure-ad-devices-in-the-azure-portal"></a>Administración de dispositivos de Azure AD en Azure Portal

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración y registro de dispositivos Protección y seguridad de la identidad

Ahora podrá encontrar todos los dispositivos conectados a Azure AD y las actividades relacionadas con dispositivos en un solo lugar. Hay una nueva experiencia de administración para administrar todas las configuraciones e identidades de dispositivos en Azure Portal. En esta versión podrá hacer lo siguiente:

- Ver todos los dispositivos que están disponibles para el acceso condicional en Azure AD.
- Ver propiedades, incluidos los dispositivos unidos a Azure AD híbrido.
- Encontrar las claves de BitLocker para los dispositivos unidos a Azure AD, administrar el dispositivo con Intune y mucho más.
- Administrar la configuración relacionada con dispositivos de Azure AD.

Para obtener más información, consulte [Administración de dispositivos con Azure Portal](../devices/device-management-azure-portal.md).

---

### <a name="support-for-macos-as-a-device-platform-for-azure-ad-conditional-access"></a>Compatibilidad con macOS como plataforma de dispositivos para el acceso condicional de Azure AD

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

Ahora puede incluir (o excluir) macOS como condición de plataforma de dispositivos en la directiva de acceso condicional de Azure AD. Con la incorporación de macOS a las plataformas de dispositivos compatibles, puede:

- **Inscribir y administrar dispositivos Mac OS con Intune.** Al igual que en otras plataformas, como iOS y Android, existe una aplicación de portal de empresa para macOS que permite realizar inscripciones unificadas. Utilice la nueva aplicación de portal de empresa para macOS para inscribir un dispositivo con Intune y registrarlo con Azure AD.
- **Asegurarse de que los dispositivos macOS se ajustan a las directivas de cumplimiento de su organización definidas en Intune.** En Intune en Azure Portal, ahora podrá configurar directivas de cumplimiento para dispositivos macOS.
- **Restringir el acceso a las aplicaciones de Azure AD a solo los dispositivos macOS que cumplan las directivas.** La creación de directivas de acceso condicional usa macOS como una opción de plataforma de dispositivos independiente. Ahora puede crear directivas de acceso condicional específicas de macOS para el conjunto de aplicaciones de destino en Azure.

Para más información, consulte:

- [Creación de una directiva de cumplimiento para dispositivos macOS con Intune](/mem/intune/protect/compliance-policy-create-mac-os)
- [Acceso condicional en Azure AD](../conditional-access/overview.md)

---

### <a name="network-policy-server-extension-for-azure-ad-multi-factor-authentication"></a>Extensión Servidor de directivas de redes para Azure AD Multi-Factor Authentication

**Tipo:** **Categoría del servicio:** Nueva característica  **Funcionalidad del producto**: Multi-Factor Authentication Autenticación de usuarios

La extensión Servidor de directivas de redes para Azure AD Multi-Factor Authentication incorpora funcionalidades de Multi-Factor Authentication basadas en la nube para su infraestructura de autenticación mediante los servidores existentes. Con la extensión Servidor de directivas de redes, podrá agregar mecanismos de verificación mediante llamadas de teléfono, mensajes de texto o aplicaciones de teléfono al flujo de autenticación existente. Para ello, no tendrá que instalar, configurar ni mantener servidores nuevos.

Esta extensión se creó para las organizaciones que quieren proteger las conexiones de redes privadas virtuales sin tener que implementar el Servidor Microsoft Azure Multi-Factor Authentication. La extensión Servidor de directivas de redes actúa como un adaptador entre RADIUS y Azure AD Multi-Factor Authentication basado en la nube para proporcionar un segundo factor de autenticación a los usuarios federados o sincronizados.

Para obtener más información, consulte [Integración de la infraestructura existente de NPS con Azure AD Multi-Factor Authentication](../authentication/howto-mfa-nps-extension.md).

---

### <a name="restore-or-permanently-remove-deleted-users"></a>Restauración o eliminación permanente de usuarios eliminados

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Administración de usuarios Directorio

En el centro de administración de Azure AD, ahora puede:

- Restaurar un usuario eliminado.
- Eliminar un usuario permanentemente.

**Para probarlo:**

1. En el centro de administración de Azure AD, seleccione [Todos los usuarios](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/UserManagementMenuBlade/All) en la sección **Administrar**.

2. En la lista **Mostrar**, seleccione **Usuarios eliminados recientemente**.

3. Seleccione uno o varios usuarios eliminados recientemente y después restáurelos o elimínelos permanentemente.

---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>Nuevas aplicaciones cliente aprobadas para el acceso condicional basado en aplicaciones de Azure AD

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

Las siguientes aplicaciones se agregaron a la lista de [aplicaciones cliente aprobadas](../conditional-access/concept-conditional-access-conditions.md#client-apps):

- Microsoft Planner
- Azure Information Protection

Para más información, consulte:

- [Requisito de aplicación cliente aprobada](../conditional-access/concept-conditional-access-conditions.md#client-apps)
- [Acceso condicional basado en aplicaciones de Azure AD](../conditional-access/app-based-conditional-access.md)

---

### <a name="use-or-between-controls-in-a-conditional-access-policy"></a>Uso del operador "OR" entre controles de una directiva de acceso condicional

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Acceso condicional Protección y seguridad de la identidad

Ahora puede usar el operador "OR" (requerir uno de los controles seleccionados) en los controles de acceso condicional. Puede usar esta característica para crear directivas con el operador "OR" entre controles de acceso. Por ejemplo, puede usar esta característica para crear una directiva que requiera que un usuario inicie sesión mediante Multi-Factor Authentication "O" que esté en un dispositivo compatible.

Para obtener más información, consulte [Controles en el acceso condicional de Azure AD](../conditional-access/controls.md).

---

### <a name="aggregation-of-real-time-risk-detections"></a>Agregación de detecciones de riesgos en tiempo real

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Identity Protection Protección y seguridad de la identidad

En Azure AD Identity Protection, todas las detecciones de riesgos en tiempo real que se han originado desde la misma dirección IP en un día determinado ahora se agregan para cada tipo de detección de riesgos. Este cambio limita el volumen de las detecciones de riesgos que se muestran sin ningún cambio en la seguridad del usuario.

La detección en tiempo real subyacente funciona cada vez que el usuario inicia sesión. Si tiene configurada una directiva de seguridad de riesgo de inicio de sesión para Multi-Factor Authentication o para bloquear el acceso, esta se desencadena durante cada inicio de sesión con riesgo.

---

## <a name="october-2017"></a>Octubre de 2017

### <a name="deprecate-azure-ad-reports"></a>Informes de Azure AD obsoletos

**Tipo:** **Categoría del servicio:** Plan de cambio **Funcionalidad del producto**: Informes Administración del ciclo de vida de la identidad

Azure Portal proporciona lo siguiente:

- Una nueva consola de administración de Azure AD.
- Nuevas API de informes de actividades y de seguridad.

Debido a estas nuevas funcionalidades, las API de informes que se encuentran en el punto de conexión /reports se retiraron el 10 de diciembre de 2017.

---

### <a name="automatic-sign-in-field-detection"></a>Detección automática de campos de inicio de sesión

**Tipo:** **Categoría del servicio:** Corregida **Funcionalidad del producto:** Mis aplicaciones Inicio de sesión único

Azure AD admite la detección automática de campos de inicio de sesión para las aplicaciones que presentan un campo de nombre de usuario y contraseña HTML. Estos pasos se documentan en [Captura automática de campos de inicio de sesión para una aplicación](../manage-apps/troubleshoot-password-based-sso.md#manually-capture-sign-in-fields-for-an-app). Puede encontrar esta funcionalidad mediante la adición de una aplicación *situada fuera de la galería* en la página **Aplicaciones empresariales** en [Azure Portal](https://aad.portal.azure.com). Además, en esta nueva aplicación podrá configurar el modo **Inicio de sesión único** en **Inicio de sesión único basado en contraseña**, especificar una URL web y, a continuación, guardar la página.

Debido a un problema del servicio, esta funcionalidad se deshabilitó temporalmente. El problema se ha resuelto y la detección automática del campo de inicio de sesión vuelve a estar disponible.

---

### <a name="new-multi-factor-authentication-features"></a>Nuevas características de Multi-Factor Authentication

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto**: Multi-Factor Authentication Protección y seguridad de la identidad

Multi-Factor Authentication (MFA) es un componente esencial para la protección de su organización. Para hacer que las credenciales tengan mayor capacidad de adaptación y que la experiencia resulte más sencilla, se han agregado las siguientes características:

- Integración de los resultados del desafío multifactor directamente en el informe de inicio de sesión de Azure AD, incluido el acceso mediante programación a los resultados de MFA.
- Integración más profunda de la configuración de MFA en la experiencia de configuración de Azure AD en Azure Portal.

Con esta versión preliminar pública, los informes y la administración de MFA son una parte integrada de la experiencia de configuración principal de AD Azure. Ahora podrá administrar la funcionalidad del portal de administración de MFA en la experiencia de Azure AD.

Para obtener más información, consulte [Referencia para los informes de la autenticación multifactor en Azure Portal](../authentication/howto-mfa-reporting.md).

---

### <a name="terms-of-use"></a>Términos de uso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Términos de uso Cumplimiento normativo

Puede usar los términos de uso de Azure AD para presentar información a los usuarios, como, por ejemplo, renuncias relevantes para los requisitos legales o de cumplimiento.

Los términos de uso de Azure AD puede utilizarse en los escenarios siguientes:

- Términos de uso generales para todos los usuarios de su organización.
- Términos de uso específicos basados en los atributos de un usuario (por ejemplo, médicos frente a enfermeras o empleados nacionales frente a internacionales, creados por grupos dinámicos).
- Términos de uso específicos para el acceso a aplicaciones empresariales de alto impacto, como, por ejemplo, Salesforce.

Para obtener más información, consulte [Términos de uso de Azure AD](../conditional-access/terms-of-use.md).

---

### <a name="enhancements-to-privileged-identity-management"></a>Mejoras en Privileged Identity Management

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Privileged Identity Management Privileged Identity Management

Con Privileged Identity Management (PIM) de Azure AD, podrá administrar, controlar y supervisar el acceso a los recursos de Azure (vista preliminar) dentro de su organización por parte de:

- Suscripciones
- Grupos de recursos
- Máquinas virtuales

Todos los recursos de Azure Portal que usan la funcionalidad RBAC de Azure pueden usar las funcionalidades de seguridad y administración del ciclo de vida que ofrece Privileged Identity Management de Azure AD.

Para obtener más información, consulte [PIM para recursos de Azure (versión preliminar)](../privileged-identity-management/azure-pim-resource-rbac.md).

---

### <a name="access-reviews"></a>Revisiones de acceso

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Revisiones de acceso Cumplimiento normativo

Las organizaciones pueden usar las revisiones de acceso (versión preliminar) para administrar de manera eficaz las pertenencias a grupos y el acceso a las aplicaciones empresariales:

- Puede volver a certificar el acceso de usuario invitado mediante las revisiones de acceso a las aplicaciones y la pertenencia a grupos. La información que proporcionan las revisiones de acceso permite a los revisores decidir de manera eficaz si deben permitir el acceso continuado a los invitados.
- Puede volver a certificar el acceso de los empleados a las aplicaciones y la pertenencia a grupos con las revisiones de acceso.

Puede recopilar los controles de revisiones de acceso en programas importantes para que su organización realice un seguimiento de las revisiones de aplicaciones vulnerables o de cumplimiento normativo.

Para obtener más información, consulte [Revisiones de acceso de Azure AD](../governance/access-reviews-overview.md).

---

### <a name="hide-third-party-applications-from-my-apps-and-the-office-365-app-launcher"></a>Ocultación de aplicaciones de terceros de Mis aplicaciones y del iniciador de aplicaciones de Office 365

**Tipo:** **Categoría del servicio:** Nueva característica **Funcionalidad del producto:** Mis aplicaciones Inicio de sesión único

Ahora podrá administrar mejor las aplicaciones que se muestran en los portales de usuario con la nueva propiedad **hide app**. Puede ocultar aplicaciones para ayudar en casos en los que se muestran iconos de aplicaciones para servicios back-end o iconos duplicados que se acumulan en los iniciadores de aplicaciones de usuarios. El botón de alternancia se encuentra en la sección **Propiedades** de la aplicación de terceros con la etiqueta **Visible to user?** (¿Visible para el usuario?). También puede ocultar una aplicación mediante programación con PowerShell.

Para obtener más información, consulte [Ocultación de una aplicación de la experiencia del usuario en Azure Active Directory](../manage-apps/hide-application-from-user-portal.md).


**¿Qué está disponible?**

 Como parte de la transición a la nueva consola de administración, hay disponibles dos nuevas API que permiten recuperar los registros de actividad de Azure AD. El nuevo conjunto de API proporciona más funciones de filtro y ordenación, así como actividades de auditoría e inicio de sesión. Los datos que estaban disponibles anteriormente a través de los informes de seguridad ahora se proporcionan mediante la API de detecciones de riesgos de Identity Protection en Microsoft Graph.


## <a name="september-2017"></a>Septiembre de 2017

### <a name="hotfix-for-identity-manager"></a>Revisión para Identity Manager

**Tipo:** **Categoría del servicio:** Característica modificada **Funcionalidad del producto:** Identity Manager Administración del ciclo de vida de la identidad

Un paquete acumulativo de revisiones (compilación 4.4.1642.0) está disponible desde el 25 de septiembre de 2017 para Microsoft Identity Manager 2016 Service Pack 1. Este paquete acumulativo de revisiones:

- Resuelve problemas e incorpora mejoras.
- Es una actualización acumulativa que reemplaza a todas las actualizaciones de Identity Manager 2016 Service Pack 1 hasta la compilación 4.4.1459.0 de Identity Manager 2016.
- Requiere disponer de Identity Manager 2016, compilación 4.4.1302.0.

Para obtener más información, consulte [Paquete acumulativo de revisiones (compilación 4.4.1642.0) está disponible para Microsoft Identity Manager SP1 de 2016](https://support.microsoft.com/help/4021562).

---
