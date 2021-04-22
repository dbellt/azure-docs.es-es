---
title: Línea base de seguridad de Azure para Azure Active Directory
description: La línea base de seguridad de Azure Active Directory proporciona una guía de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: active-directory
ms.topic: conceptual
ms.date: 04/07/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 8d4203b7b5d7742bf198778864fa4f42b7423596
ms.sourcegitcommit: c6a2d9a44a5a2c13abddab932d16c295a7207d6a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107286009"
---
# <a name="azure-security-baseline-for-azure-active-directory"></a>Línea base de seguridad de Azure para Azure Active Directory

Esta línea base de seguridad aplica las instrucciones de la [versión 2.0 de Azure Security Benchmark](../../security/benchmarks/overview.md) a Azure Active Directory. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa por medio de los **controles de seguridad** que define Azure Security Benchmark y las directrices aplicables a Azure Active Directory relacionadas. 

> [!NOTE]
> Se han excluido de esta línea base los **controles** que no son aplicables a Azure Active Directory o cuya responsabilidad es de Microsoft. Para ver cómo Azure Active Directory se asigna por completo a Azure Security Benchmark, consulte el [archivo completo de asignación de línea base de seguridad de Azure Active Directory](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Seguridad de redes

*Para más información, consulte [Azure Security Benchmark: seguridad de red](/azure/security/benchmarks/security-controls-v2-network-security).*

### <a name="ns-6-simplify-network-security-rules"></a>NS-6: simplificación de las reglas de seguridad de red

**Guía**: Use etiquetas de servicio de Azure Virtual Network para definir controles de acceso a la red en los grupos de seguridad de red o las instancias de Azure Firewall configuradas para los recursos de Azure Active Directory. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio "AzureActiveDirectory" en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian. 

- [Descripción y uso de etiquetas de servicio](../../virtual-network/service-tags-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="identity-management"></a>Administración de identidades

*Para más información, consulte [Azure Security Benchmark: Administración de identidades](/azure/security/benchmarks/security-controls-v2-identity-management).*

### <a name="im-1-standardize-azure-active-directory-as-the-central-identity-and-authentication-system"></a>IM-1: Unificación en Azure Active Directory como sistema central de identidad y autenticación

**Guía**: Azure Migrate usa Azure Active Directory (Azure AD) como un servicio de administración de identidades y acceso predeterminado de Azure. Debe unificar Azure AD para controlar la administración de identidades y accesos de la organización en: 
- Los recursos de Microsoft Cloud, como Azure Portal, Azure Storage, Azure Virtual Machines (Linux y Windows), Azure Key Vault, PaaS y aplicaciones SaaS. 
- Los recursos de su organización, como aplicaciones en Azure o los recursos de la red corporativa. 
 

La protección de Azure AD debe tener máxima prioridad en la práctica de seguridad en la nube de su organización. Azure AD proporciona una puntuación de seguridad de la identidad para ayudarle a evaluar la posición de seguridad de las identidades en relación con los procedimientos recomendados de Microsoft. Use la puntuación para medir el grado de coincidencia de la configuración con los procedimientos recomendados y para hacer mejoras en la posición de seguridad. 

Azure AD admite identidades externas que permiten a los usuarios que no tienen una cuenta de Microsoft iniciar sesión en sus aplicaciones y recursos mediante la identidad externa. 

- [Inquilinos en Azure Active Directory](../develop/single-and-multi-tenant-apps.md)

- [Procedimiento para crear y configurar una instancia de Azure AD](active-directory-access-create-new-tenant.md)

- [Uso de proveedores de identidades externos para una aplicación](/azure/active-directory/b2b/identity-providers)

- [¿Qué es la puntuación de seguridad de la identidad en Azure Active Directory?](identity-secure-score.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-2-manage-application-identities-securely-and-automatically"></a>IM-2: Administración de identidades de aplicaciones de forma segura y automática

**Guía**: Use Identidad administrada para los recursos de Azure para cuentas que no son humanas, como servicios o automatización. Se recomienda usar la característica de identidad administrada de Azure en lugar de crear una cuenta humana más eficaz para acceder a los recursos o ejecutarlos. Puede autenticarse de forma nativa en los servicios o recursos de Azure que admiten la autenticación de Azure Active Directory (Azure AD) mediante una regla de concesión de acceso predefinida sin usar la credencial codificada de forma rígida en el código fuente o los archivos de configuración. No se pueden asignar identidades administradas de Azure a recursos de Azure AD, Azure AD es el mecanismo para autenticarse con identidades administradas asignadas a los recursos de otro servicio.

- [Identidad administrada para recursos de Azure](../managed-identities-azure-resources/overview.md)

- [Servicios que admiten identidades administradas para recursos de Azure](../managed-identities-azure-resources/services-support-managed-identities.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-3-use-azure-ad-single-sign-on-sso-for-application-access"></a>IM-3: Uso del inicio de sesión único de Azure AD para acceder a las aplicaciones

**Guía**: Use Azure Active Directory para proporcionar administración de identidades y accesos a recursos de Azure, aplicaciones en la nube y aplicaciones locales. Esto incluye no solo las identidades empresariales, como los empleados, sino también las identidades externas, como asociados y proveedores. Esto permite que el inicio de sesión único (SSO) administre y proteja el acceso a los datos y recursos de la organización locales y en la nube. Conecte todos los usuarios, las aplicaciones y los dispositivos a Azure AD para obtener un acceso seguro y sin problemas, y para lograr mayor visibilidad y control.

 
- [Descripción del inicio de sesión único de aplicaciones con Azure AD](../manage-apps/what-is-single-sign-on.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-4-use-strong-authentication-controls-for-all-azure-active-directory-based-access"></a>IM-4: Uso de controles con autenticación multifactor sólida para todo el acceso basado en Azure Active Directory

**Guía**: Use Azure Active Directory para admitir controles de autenticación sólida a través de la autenticación multifactor (MFA) y otros métodos seguros sin contraseña.
- Autenticación multifactor: habilite Azure AD MFA y siga las recomendaciones de administración de identidades y acceso de Azure Security Center para conocer varios procedimientos recomendados en la configuración de la autenticación multifactor. La autenticación multifactor se puede exigir a todos los usuarios, a usuarios concretos o a nivel de cada usuario en función de los factores de riesgo y las condiciones de inicio de sesión.
- Autenticación sin contraseña: hay disponibles tres opciones de autenticación sin contraseña: Windows Hello para empresas, aplicación Microsoft Authenticator y métodos de autenticación locales, como las tarjetas inteligentes.

Para administradores y usuarios con privilegios, asegúrese de que se usa el nivel más alto del método de autenticación sólida, seguido de la implementación de la directiva de autenticación sólida adecuada para otros usuarios.

Azure AD admite la autenticación heredada basada en contraseña como las cuentas solo para la nube (es decir, cuentas de usuario creadas directamente en Azure AD) que tienen una directiva de contraseñas de línea base o las cuentas híbridas (cuentas de usuario que provienen de instancias de Azure Active Directory locales) que siguen las directivas de contraseñas locales. Cuando se usa la autenticación con contraseña, Azure AD proporciona una funcionalidad de protección de contraseña que impide que los usuarios establezcan contraseñas fáciles de adivinar. Microsoft proporciona una lista global de contraseñas prohibidas que se actualiza en función de la telemetría, y los clientes pueden ampliarla en función de sus necesidades (por ejemplo, personalización de marca, referencias culturales, etc.). Esta protección de contraseñas se puede usar para las cuentas híbridas y solo en la nube.

La autenticación basada solo en las credenciales de contraseña es susceptible a métodos de ataque populares. Para una mayor seguridad, use una autenticación sólida, como MFA y una directiva de contraseñas segura. En el caso de las aplicaciones de terceros y los servicios de Marketplace que pueden tener contraseñas predeterminadas, debe cambiarlas durante la configuración inicial del servicio.

 
- [Plan de una implementación de Azure AD Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md) 

 

 
- [Introducción a las opciones de autenticación sin contraseña de Azure Active Directory](../authentication/concept-authentication-passwordless.md) 

 

 
- [Directiva de contraseñas predeterminada de Azure AD](https://docs.microsoft.com/azure/active-directory/authentication/concept-sspr-policy#password-policies-that-only-apply-to-cloud-user-accounts)

- [Eliminación de contraseñas incorrectas mediante la Protección con contraseña de Azure AD](../authentication/concept-password-ban-bad.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-5-monitor-and-alert-on-account-anomalies"></a>IM-5: Supervisión y alerta de anomalías de cuenta

**Guía**: Azure Active Directory proporciona los siguientes orígenes de datos:

 
- Inicios de sesión: el informe de los inicios de sesión proporciona información sobre el uso de aplicaciones administradas y las actividades de inicio de sesión del usuario.

 
- Los registros de auditoría proporcionan rastreabilidad mediante los registros de todos los cambios realizados por diversas características de Azure AD. Algunos ejemplos de registros de auditoría incluyen los cambios realizados en cualquier recurso de Azure AD como agregar o quitar usuarios, aplicaciones, grupos, roles y directivas.

 
- Inicios de sesión de riesgo: un inicio de sesión de riesgo es un indicador de un intento de inicio de sesión que puede haber realizado alguien que no es el propietario legítimo de una cuenta de usuario.

 
- Usuarios marcados en riesgo: un usuario en riesgo es un indicador de una cuenta de usuario que puede haber estado en peligro.

 

Estos orígenes de datos se pueden integrar con Azure Monitor, Azure Sentinel o sistemas SIEM de terceros.

 

 
Azure Security Center también puede alertar sobre determinadas actividades sospechosas, como un número excesivo de intentos de autenticación incorrectos y cuentas en desuso en la suscripción.

 

 
Azure Advanced Threat Protection (AATP) es una solución de seguridad que puede usar señales de Active Directory para identificar, detectar e investigar amenazas avanzadas, identidades en riesgo y acciones internas malintencionadas.

 

 
- [Informes de actividad de auditoría en Azure Active Directory](../reports-monitoring/concept-audit-logs.md) 

 

 
- [Visualización de los inicios de sesión de riesgo de Azure AD](/azure/active-directory/reports-monitoring/concept-risky-sign-ins) 

 

 
- [Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo](/azure/active-directory/reports-monitoring/concept-user-at-risk) 

 

 
- [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](../../security-center/security-center-identity-access.md) 

 

 
- [Alertas del módulo de protección de inteligencia sobre amenazas de Azure Security Center](../../security-center/alerts-reference.md) 

 

 
- [Integración de los registros de actividad de Azure en Azure Monitor](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-6-restrict-azure-resource-access-based-on-conditions"></a>IM-6: Restricción del acceso a recursos de Azure en función de las condiciones

**Guía**: Use el acceso condicional de Azure Active Directory (Azure AD) para un control de acceso más granular basado en condiciones definidas por el usuario, por ejemplo, los inicios de sesión de usuario desde determinados intervalos IP tendrán que usar MFA para iniciar sesión. También se puede usar la directiva de administración de sesión de autenticación granular para distintos casos de uso.

 
- [¿Qué es el acceso condicional?](../conditional-access/overview.md) 

 

 
- [Directivas de acceso condicional habituales](../conditional-access/concept-conditional-access-policy-common.md) 

 

 
- [Frecuencia de inicio de sesión de usuario](../conditional-access/howto-conditional-access-session-lifetime.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-8-secure-user-access-to-legacy-applications"></a>IM-8: Protección del acceso de los usuarios a las aplicaciones heredadas

**Guía**: Asegúrese de que tiene controles de acceso modernos y supervisión de sesiones para las aplicaciones heredadas y los datos que almacenan y procesan. Mientras que las VPN suelen usarse para acceder a las aplicaciones heredadas, a menudo tienen un control de acceso básico y una supervisión de sesión limitada.

 
Azure AD Application Proxy permite publicar aplicaciones locales heredadas en usuarios remotos con el inicio de sesión único mientras se valida explícitamente la confiabilidad de los usuarios y dispositivos remotos con el acceso condicional de Azure AD.

 

 
Como alternativa, Microsoft Cloud App Security es un servicio de agente de seguridad de acceso a la nube (CASB) que puede proporcionar controles para supervisar las sesiones de aplicación de un usuario y las acciones de bloqueo, para aplicaciones locales heredadas y aplicaciones de software como servicio (SaaS) en la nube.

 

 
- [Azure Active Directory Application Proxy](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy#what-is-application-proxy) 

 

 
- [Procedimientos recomendados de Microsoft Cloud App Security](/cloud-app-security/best-practices)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="privileged-access"></a>Acceso con privilegios

*Para más información, consulte [Azure Security Benchmark: Acceso con privilegios](/azure/security/benchmarks/security-controls-v2-privileged-access).*

### <a name="pa-1-protect-and-limit-highly-privileged-users"></a>PA-1: Protección y limitación de usuarios con privilegios elevados

**Guía**: Los roles integrados más críticos de Azure AD son los de administrador global y administrador de roles con privilegios, ya que los usuarios que tengan asignados estos dos roles pueden delegar roles de administrador:

- Administrador global: los usuarios con este rol tienen acceso a todas las características administrativas de Azure AD, así como a los servicios que utilizan identidades de Azure AD.

- Administrador de roles con privilegios: los usuarios con este rol pueden administrar asignaciones de roles en Azure AD, y dentro de Azure AD Privileged Identity Management (PIM). Además, este rol permite administrar todos los aspectos de PIM y de las unidades administrativas.

Si usa roles personalizados con determinados permisos con privilegios asignados, es posible que tenga otros roles críticos que deban administrarse. Además, puede que también quiera aplicar controles similares a la cuenta de administrador de recursos empresariales críticos.

Azure AD cuenta con privilegios elevados: los usuarios y entidades de servicio que están asignados directa o indirectamente a los roles de administrador global o administrador de roles con privilegios, o que son elegibles para ellos, y otros roles con privilegios elevados en Azure AD y Azure.

Limite el número de cuentas con privilegios elevados y proteja estas cuentas en un nivel elevado, ya que los usuarios con este privilegio pueden leer directa o indirectamente, y modificar todos los recursos de su entorno de Azure.

Puede habilitar el acceso con privilegios Just-in-Time (JIT) a los recursos de Azure y Azure AD mediante Azure AD Privileged Identity Management (PIM). JIT concede permisos temporales para realizar tareas con privilegios solo cuando los usuarios lo necesitan. PIM también puede generar alertas de seguridad cuando hay actividades sospechosas o no seguras en la organización de Azure AD.

- [Permisos de rol administrativo en Azure AD](/azure/active-directory/users-groups-roles/directory-assign-admin-roles) 

- [Uso de alertas de seguridad de Azure Privileged Identity Management](../privileged-identity-management/pim-how-to-configure-security-alerts.md) 

- [Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD](/azure/active-directory/users-groups-roles/directory-admin-roles-secure)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-2-restrict-administrative-access-to-business-critical-systems"></a>PA-2: Restricción del acceso administrativo a los sistemas críticos para la empresa

**Guía**: Use Azure Active Directory Privileged Identity Management y la autenticación multifactor para restringir el acceso administrativo a los sistemas críticos para la empresa.

- [Aprobación o rechazo de solicitudes para los roles de Azure AD en Privileged Identity Management](../privileged-identity-management/azure-ad-pim-approval-workflow.md)

- [Acceso condicional: Exigir autenticación multifactor para administradores](../conditional-access/howto-conditional-access-policy-admin-mfa.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-3-review-and-reconcile-user-access-regularly"></a>PA-3: Revisión y conciliación de manera periódica del acceso de los usuarios

**Guía**: Revise periódicamente las asignaciones de acceso de cuentas de usuario para asegurarse de que las cuentas y su acceso son válidos, especialmente centrados en los roles con privilegios elevados, incluidos el de administrador global y el administrador de roles con privilegios. Puede usar las revisiones de Azure Active Directory (Azure AD) para revisar las pertenencias a grupos, el acceso a aplicaciones empresariales y las asignaciones de roles, tanto para roles de Azure AD como para roles de Azure. Los informes de Azure AD pueden proporcionar registros para ayudar a detectar cuentas obsoletas. También puede usar Azure AD Privileged Identity Management para crear un flujo de trabajo de informes de revisión de acceso para facilitar el proceso de revisión.

Además, se puede configurar Azure Privileged Identity Management para enviar una alerta cuando se cree un número de cuentas de administrador excesivo y para identificar las cuentas de administrador que hayan quedado obsoletas o que no estén configuradas correctamente.

- [Creación de una revisión de acceso para los roles de Azure AD en Azure AD Privileged Identity Management (PIM)](../privileged-identity-management/pim-how-to-start-security-review.md)

- [Creación de una revisión de acceso de los roles de recursos de Azure en Privileged Identity Management (PIM)](../privileged-identity-management/pim-resource-roles-start-access-review.md)

- [Procedimiento para usar las revisiones de acceso e identidades de Azure AD](../governance/access-reviews-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-4-set-up-emergency-access-in-azure-ad"></a>PA-4: Configuración del acceso de emergencia en Azure AD

**Guía**: Para evitar que se le bloquee por accidente el acceso a la organización de Azure AD, configure una cuenta de acceso de emergencia para cuando no se puedan usar cuentas administrativas normales. Las cuentas de acceso de emergencia tienen normalmente privilegios elevados y no se asignan a usuarios específicos.
Las cuentas de acceso de emergencia se limitan a situaciones "excepcionales" o de emergencia en las que no se pueden usar las cuentas administrativas normales.

Debe asegurarse de que las credenciales (como contraseña, certificado o tarjeta inteligente) de las cuentas de acceso de emergencia estén protegidas y solo las conozcan aquellas personas que estén autorizadas a usarlas solo en caso de emergencia.

- [Administración de cuentas de acceso de emergencia en Azure AD](/azure/active-directory/users-groups-roles/directory-emergency-access)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-5-automate-entitlement-management"></a>PA-5: Automatización de la administración de derechos 

**Guía**: Use las características de administración de derechos de Azure AD para automatizar los flujos de trabajo de solicitud de acceso, como las asignaciones, las revisiones y la expiración del acceso. También se admite la aprobación en dos o varias fases.

- [¿Qué es la administración de derechos de Azure AD?](../governance/entitlement-management-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-6-use-privileged-access-workstations"></a>PA-6: Uso de estaciones de trabajo con privilegios de acceso

**Guía**: Las estaciones de trabajo seguras y aisladas son de una importancia vital para la seguridad de los roles con acceso a información confidencial como administradores, desarrolladores y operadores de servicios críticos. Use estaciones de trabajo de usuario de alta seguridad o Azure Bastion para las tareas administrativas. Use Azure Active Directory, Protección contra amenazas avanzada de Microsoft Defender (ATP) o Microsoft Intune para implementar una estación de trabajo de usuario segura y administrada para las tareas administrativas. Las estaciones de trabajo protegidas se pueden administrar de forma centralizada para aplicar una configuración segura, como autenticación sólida, líneas de base de software y hardware y acceso lógico y de red restringido.

- [Protección de dispositivos como parte de la historia de acceso con privilegios](/security/compass/privileged-access-devices)

- [Implementación del acceso con privilegios](/security/compass/privileged-access-deployment)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-7-follow-just-enough-administration-least-privilege-principle"></a>PA-7: Seguimiento de solo una administración suficiente (principio de privilegios mínimos) 

**Guía**: Los clientes pueden configurar su implementación de Azure Active Directory (Azure AD) con privilegios mínimos, mediante la asignación de usuarios a los roles con los permisos mínimos necesarios para que los usuarios completen sus tareas administrativas.

- [Permisos de rol administrativo en Azure AD](../roles/permissions-reference.md)

- [Asignación de roles de Azure AD a usuarios](../roles/manage-roles-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-8-choose-approval-process-for-microsoft-support"></a>PA-8: Selección del proceso de aprobación para el soporte técnico de Microsoft  

**Guía**: Azure Active Directory no admite la caja de seguridad del cliente. Microsoft puede trabajar con los clientes mediante métodos que no son de caja de seguridad para la aprobación del acceso a los datos de los clientes.

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

## <a name="data-protection"></a>Protección de datos

*Para más información, consulte [Azure Security Benchmark: protección de datos](/azure/security/benchmarks/security-controls-v2-data-protection).*

### <a name="dp-2-protect-sensitive-data"></a>DP-2: Protección de datos confidenciales

**Guía**: Tenga en cuenta las siguientes instrucciones para implementar la protección de los datos confidenciales:

- **Aislamiento**: un directorio es el límite de datos por el que los servicios de Azure Active Directory (Azure AD) almacenan y procesan los datos de un cliente. Los clientes deben determinar dónde desean que resida la mayoría de los datos de Azure AD del cliente mediante el establecimiento de la propiedad Country en su directorio.

- **Segmentación**: el rol del administrador global tiene control total de todos los datos de directorio y las reglas que rigen el acceso y las instrucciones de procesamiento. Un directorio se puede segmentar en unidades administrativas y aprovisionarse con usuarios y grupos que administrarán los administradores de esas unidades. Los administradores globales pueden delegar la responsabilidad a otros principios de su organización al asignarles a roles predefinidos o roles personalizados que puedan crear.

 
- **Acceso:** los permisos se pueden aplicar en un usuario, grupo, rol, aplicación o dispositivo. La asignación puede ser permanente o temporal, según la configuración de Privileged Identity Management. 
  
- **Cifrado**: Azure AD cifra todos los datos en reposo o en tránsito. La oferta no permite a los clientes cifrar datos de directorio con su propia clave de cifrado. 

Para determinar cómo se asigna el país seleccionado a la ubicación física del directorio, consulte el artículo "Dónde se encuentran tus datos".

- [Dónde se encuentran tus datos](https://www.microsoft.com/trust-center/privacy/data-location)

Como el cliente usa herramientas, características y aplicaciones de Azure AD que interactúan con su directorio, utilice el artículo Azure Active Directory: ¿Dónde se encuentran los datos?

- [¿Dónde se encuentran los datos?](https://msit.powerbi.com/view?r=eyJrIjoiYzEyZTc5OTgtNTdlZS00ZTVkLWExN2ItOTM0OWU4NjljOGVjIiwidCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsImMiOjV9)

- [Documentación de los roles de Azure AD](/azure/active-directory/roles/)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="dp-4-encrypt-sensitive-information-in-transit"></a>DP-4: Cifrado de la información confidencial en tránsito

**Guía**: Para complementar los controles de acceso, los datos en tránsito deben protegerse frente a ataques de "fuera de banda" (por ejemplo, captura del tráfico) mediante cifrado para que los atacantes no puedan leer ni modificar los datos fácilmente.

Azure AD admite el cifrado de datos en tránsito con TLS v1.2 u otra versión posterior.

Aunque esto es opcional en el caso del tráfico de redes privadas, es fundamental para el tráfico de redes externas y públicas. Asegúrese de que los clientes que se conectan a los recursos de Azure puedan negociar TLS v1.2 o superior. Para la administración remota, use SSH (para Linux) o RDP/TLS (para Windows) en lugar de un protocolo sin cifrar. Se deben deshabilitar los protocolos y las versiones de SSL, TLS y SSH obsoletos, así como los cifrados débiles.

De forma predeterminada, Azure proporciona el cifrado de los datos en tránsito entre los centros de datos de Azure.

- [Descripción del cifrado en tránsito con Azure](https://docs.microsoft.com/azure/security/fundamentals/encryption-overview#encryption-of-data-in-transit) 

- [Información sobre la seguridad de TLS](/security/engineering/solving-tls1-problem) 

- [Cifrado doble para datos de Azure en tránsito](https://docs.microsoft.com/azure/security/fundamentals/double-encryption#data-in-transit)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="asset-management"></a>Administración de recursos

*Para más información, consulte [Azure Security Benchmark: Administración de recursos](/azure/security/benchmarks/security-controls-v2-asset-management).*

### <a name="am-1-ensure-security-team-has-visibility-into-risks-for-assets"></a>AM-1: Asegurarse de que el equipo de seguridad tiene visibilidad sobre los riesgos para los recursos

**Guía**: Asegúrese de que se conceden a los equipos de seguridad permisos de lector de seguridad en el inquilino y las suscripciones de Azure para que puedan supervisar los riesgos de seguridad mediante Azure Security Center. 

En función de la estructura de las responsabilidades del equipo de seguridad, la supervisión de los riesgos de seguridad puede ser responsabilidad de un equipo de seguridad central o de un equipo local. Dicho esto, la información y los riesgos de seguridad siempre se deben agregar de forma centralizada dentro de una organización. 

Los permisos de lector de seguridad se pueden aplicar en general a un inquilino completo (grupo de administración raíz) o a grupos de administración o a suscripciones específicas. 

Es posible que se requieran permisos adicionales para obtener visibilidad de las cargas de trabajo y los servicios. 

- [Introducción al rol de lector de seguridad](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#security-reader)

- [Información general sobre los grupos de administración de Azure](../../governance/management-groups/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="am-5-limit-users-ability-to-interact-with-azure-resource-manager"></a>AM-5: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Guía**: use el acceso condicional de Azure Active Directory (Azure AD) para limitar la capacidad de los usuarios de interactuar con Azure AD mediante Azure Portal al configurar la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure".

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../../role-based-access-control/conditional-access-azure-management.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="logging-and-threat-detection"></a>registro y detección de amenazas

*Para más información, consulte [Azure Security Benchmark: registro y detección de amenazas](/azure/security/benchmarks/security-controls-v2-logging-threat-protection).*

### <a name="lt-1-enable-threat-detection-for-azure-resources"></a>LT-1: Habilitación de la detección de amenazas para recursos de Azure

**Guía**: Use la funcionalidad de Azure Active Directory (Azure AD) Identity Protection integrada para la detección de amenazas para los recursos de Azure AD. 
 
 
 

 
Azure AD genera registros de actividad que a menudo se usan para la detección o búsqueda de amenazas. Los registros de inicio de sesión de Azure AD proporcionan un registro de la actividad de autenticación y autorización para usuarios, servicios y aplicaciones. Los registros de auditoría de Azure AD realizan un seguimiento de los cambios realizados en un inquilino de Azure AD, incluidos los cambios que mejoran o disminuyen la posición de seguridad.

- [¿Qué es Azure Active Directory Identity Protection?](../identity-protection/overview-identity-protection.md)

- [Conexión de datos de Azure AD Identity Protection a Azure Sentinel.](../../sentinel/connect-azure-ad-identity-protection.md)

- [Conexión de datos de Azure Active Directory (Azure AD) a Azure Sentinel](../../sentinel/connect-azure-active-directory.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="lt-2-enable-threat-detection-for-azure-identity-and-access-management"></a>LT-2: Habilitación de la detección de amenazas para la administración de identidades y acceso de Azure

**Guía**: Azure Active Directory (Azure AD) proporciona los siguientes registros de usuario que se pueden ver en los informes de Azure AD o integrados en Azure Monitor, Azure Sentinel, SIEM o herramientas de supervisión para casos de uso de supervisión y análisis más sofisticados: 
- Inicios de sesión: el informe de inicios de sesión proporciona información sobre el uso de aplicaciones administradas y actividades de inicio de sesión de usuario. 
- Los registros de auditoría proporcionan rastreabilidad mediante los registros de todos los cambios realizados por diversas características de Azure AD. Algunos ejemplos de registros de auditoría incluyen los cambios realizados en cualquier recurso de Azure AD como agregar o quitar usuarios, aplicaciones, grupos, roles y directivas. 
- Inicios de sesión de riesgo: un inicio de sesión de riesgo es un indicador de un intento de inicio de sesión que puede haber realizado alguien que no es el propietario legítimo de una cuenta de usuario. 
- Usuarios de riesgo: un usuario de riesgo es un indicador de una cuenta de usuario que puede haber estado en peligro. 

Las detecciones de riesgos de Identity Protection están habilitadas de forma predeterminada y no requieren ningún proceso de incorporación. Los datos de granularidad o riesgo se determinan mediante la SKU de licencia. 

- [Informes de actividad de auditoría en Azure Active Directory](../reports-monitoring/concept-audit-logs.md)  

- [Habilitación de Azure Identity Protection](../identity-protection/overview-identity-protection.md)  

- [Protección contra amenazas en Azure Security Center](/azure/security-center/threat-protection)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="lt-4-enable-logging-for-azure-resources"></a>LT-4: Habilitación del registro para recursos de Azure

**Guía**: Azure Active Directory (Azure AD) genera registros de actividad. A diferencia de algunos servicios de Azure, Azure AD distingue entre los registros de actividad y de recursos. Los registros de actividad están disponibles automáticamente en la sección de Azure AD de Azure Portal y se pueden exportar a Azure Monitor, Azure Event Hubs, Azure Storage, SIEMs y otras ubicaciones.
 
- Inicios de sesión: el informe de inicios de sesión proporciona información sobre el uso de aplicaciones administradas y actividades de inicio de sesión de usuario.  
 
- Los registros de auditoría proporcionan rastreabilidad mediante los registros de todos los cambios realizados por diversas características de Azure AD. Algunos ejemplos de registros de auditoría incluyen los cambios realizados en cualquier recurso de Azure AD como agregar o quitar usuarios, aplicaciones, grupos, roles y directivas.  
 
 

Azure AD también proporciona registros relacionados con la seguridad que se pueden ver en los informes de Azure AD o que están integrados con Azure Monitor, Azure Sentinel u otras herramientas de SIEM o supervisión para casos más sofisticados de uso de supervisión y análisis: 
- Inicios de sesión de riesgo: un inicio de sesión de riesgo es un indicador de un intento de inicio de sesión que puede haber realizado alguien que no es el propietario legítimo de una cuenta de usuario.
- Usuarios de riesgo: un usuario de riesgo es un indicador de una cuenta de usuario que puede haber estado en peligro. 

Las detecciones de riesgos de Identity Protection están habilitadas de forma predeterminada y no requieren ningún proceso de incorporación. Los datos de granularidad o riesgo se determinan mediante la SKU de licencia. 

 
- [¿Qué son los informes de Azure Active Directory?](../reports-monitoring/overview-reports.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="lt-5-centralize-security-log-management-and-analysis"></a>LT-5: Centralizar la administración y el análisis de los registros de seguridad

**Guía**: Se recomiendan las siguientes directrices cuando los clientes quieran centralizar sus registros de seguridad para facilitar la búsqueda de amenazas y el análisis de la posición de seguridad:
 
- Centralice el almacenamiento y el análisis de los registros para permitir su correlación. Asegúrese de que, para cada origen de registro en Azure AD, asigna un propietario de datos, una guía de acceso y una ubicación de almacenamiento; que determinar qué herramientas se usan para procesar y acceder a los datos, y los requisitos de retención de datos.  
 
- Asegúrese también de que integra los registros de actividad de Azure en el registro central. Recopile registros mediante Azure Monitor para agregar datos de seguridad generados por dispositivos de punto de conexión, recursos de red y otros sistemas de seguridad. En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.  
 
- Además, habilite e incorpore los datos a Azure Sentinel o a un sistema SIEM de terceros.  
 

Muchas organizaciones optan por usar Azure Sentinel para los datos de acceso frecuente y Azure Storage para los datos inactivos que se usan con menos frecuencia.  
 

- [Recopilación de registros y métricas de plataforma con Azure Monitor](/azure/azure-monitor/platform/diagnostic-settings)   
 

- [Incorporación de Azure Sentinel](../../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="lt-6-configure-log-storage-retention"></a>LT-6: Configuración de la retención del almacenamiento de registros

**Guía**: Asegúrese de que todas las cuentas de almacenamiento o las áreas de trabajo de Log Analytics que se usan para almacenar registros de inicio de sesión de Azure Active Directory, registros de auditoría y registros de datos de riesgo tengan el período de retención de registros configurado de acuerdo a la normativa de cumplimiento de la organización.

En Azure Monitor, puede establecer el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización. Use cuentas de área de trabajo de Azure Storage, Data Lake o Log Analytics para el almacenamiento de archivo a largo plazo.

- [Configuración del período de retención del área de trabajo de Log Analytics](/azure/azure-monitor/platform/manage-cost-storage)  

- [Almacenamiento de registros de recursos en una cuenta de Azure Storage](/azure/azure-monitor/platform/resource-logs-collect-storage)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Azure Security Benchmark: respuesta ante incidentes](/azure/security/benchmarks/security-controls-v2-incident-response).*

### <a name="ir-1-preparation--update-incident-response-process-for-azure"></a>IR-1: Preparación: actualización del proceso de respuesta a incidentes para Azure

**Guía**: Asegúrese de que la organización tenga procesos para responder a incidentes de seguridad, que haya actualizado estos procesos para Azure y que los ejercite regularmente para garantizar su disponibilidad.

- [Implementación de la seguridad en todo el entorno empresarial](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

- [Guía de referencia de respuesta a incidentes](/microsoft-365/downloads/IR-Reference-Guide.pdf)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="ir-2-preparation--setup-incident-notification"></a>IR-2: Preparación: notificación de incidentes de configuración

**Guía**: Configure la información de contacto en caso de incidentes de seguridad en Azure Security Center. Microsoft utilizará esta información para ponerse en contacto con usted si el Centro de respuestas de seguridad de Microsoft (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a sus datos. También hay opciones para personalizar la alerta y notificación de incidentes en diferentes servicios de Azure en función de sus necesidades de respuesta a incidentes. 

- [Establecimiento del contacto de seguridad de Azure Security Center](../../security-center/security-center-provide-security-contact-details.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="ir-3-detection-and-analysis--create-incidents-based-on-high-quality-alerts"></a>IR-3: Detección y análisis: creación de incidentes en función de alertas de alta calidad 

**Guía**: Asegúrese de que cuenta con un proceso para crear alertas de alta calidad y medir la calidad de las alertas. Esto le permite aprender de incidentes anteriores y clasificar las alertas para los analistas, de modo que no pierdan tiempo con falsos positivos. 

Las alertas de alta calidad se pueden crear en función de la experiencia de pasados incidentes, información validada procedente de la comunidad y herramientas diseñadas para generar y limpiar alertas mediante la fusión y correlación de diversos orígenes de la señal. 

Azure Security Center proporciona alertas de alta calidad en muchos recursos de Azure. Puede usar el conector de datos de ASC para enviar las alertas a Azure Sentinel. Azure Sentinel le permite crear reglas de alerta avanzadas con el fin de generar automáticamente incidentes para investigar. 

Exporte las alertas y recomendaciones de Azure Security Center mediante la característica de exportación para ayudar a identificar riesgos en los recursos de Azure. Esta exportación puede hacerse de forma manual o de modo continuo.

- [Configuración de la exportación](../../security-center/continuous-export.md)

- [Transmisión de alertas a Azure Sentinel](../../sentinel/connect-azure-security-center.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="ir-4-detection-and-analysis--investigate-an-incident"></a>IR-4: Detección y análisis: investigación de incidentes

**Guía**: Asegúrese de que los analistas pueden consultar y usar diversos orígenes de datos a medida que investigan posibles incidentes, para conseguir una visión global de lo que ha sucedido. Se deben recopilar diversos registros para realizar un seguimiento de las actividades de un posible atacante en la cadena de eliminación para evitar puntos ciegos.  También debe asegurarse de que se capturan detalles y aprendizajes para otros analistas y para futuras referencias históricas.  

Los orígenes de datos para la investigación incluyen los orígenes de registro centralizados que ya se recopilan de los servicios en el ámbito y los sistemas en ejecución, pero también pueden incluir:

- Datos de red: use registros de flujo de grupos de seguridad de red, Azure Network Watcher y Azure Monitor para capturar registros de flujo de red y otra información de análisis. 

- Instantáneas de sistemas en ejecución: 

    - Use la funcionalidad de instantáneas de la máquina virtual de Azure para crear una instantánea del disco del sistema en ejecución. 

    - Use la funcionalidad de volcado de la memoria nativa del sistema operativo para crear una instantánea de la memoria del sistema en ejecución.

    - Use la característica de instantánea de los servicios de Azure o la propia funcionalidad del software para crear instantáneas de los sistemas en ejecución.

Azure Sentinel proporciona amplios análisis de datos en prácticamente cualquier origen de registro y un portal de administración de casos para administrar todo el ciclo de vida de los incidentes. La información de inteligencia durante una investigación puede asociarse a un incidente con fines de seguimiento e informes. 

- [Instantánea del disco de una máquina Windows](../../virtual-machines/windows/snapshot-copy-managed-disk.md)

- [Instantánea del disco de una máquina Linux](../../virtual-machines/linux/snapshot-copy-managed-disk.md)

- [Recopilación del volcado de memoria e información de diagnóstico del servicio de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) 

- [Investigación de incidentes con Azure Sentinel](../../sentinel/tutorial-investigate-cases.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="ir-5-detection-and-analysis--prioritize-incidents"></a>IR-5: Detección y análisis: clasificar incidentes

**Guía**: Proporcione contexto a los analistas sobre los incidentes en los que deberán centrarse en primer lugar en función de la gravedad de la alerta y la confidencialidad de los recursos. 

Azure Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Adicionalmente, marque los recursos con etiquetas y cree un sistema de nomenclatura para identificar y clasificar los recursos de Azure, especialmente los que procesan datos confidenciales.  Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente.

- [Alertas de seguridad en el Centro de seguridad de Azure](../../security-center/security-center-alerts-overview.md)

- [Uso de etiquetas para organizar los recursos de Azure](/azure/azure-resource-manager/resource-group-using-tags)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="ir-6-containment-eradication-and-recovery--automate-the-incident-handling"></a>IR-6: Contención, erradicación y recuperación: automatización del control de incidentes

**Guía**: Automatice las tareas repetitivas manuales para acelerar el tiempo de respuesta y reducir la carga de los analistas. Las tareas manuales tardan más tiempo en ejecutarse, lo que ralentiza cada incidente y reduce el número de incidentes que un analista puede manejar. Las tareas manuales también aumentan la fatiga del analista, lo que aumenta el riesgo del error humano que produce retrasos y reduce la capacidad de los analistas de centrarse de manera efectiva en tareas complejas. Use las características de automatización de flujo de trabajo de Azure Security Center y Azure Sentinel para desencadenar acciones automáticamente o ejecutar un cuaderno de estrategias para responder a las alertas de seguridad entrantes. El cuaderno de estrategias lleva a cabo acciones, como el envío de notificaciones, la deshabilitación de cuentas y el aislamiento de redes problemáticas. 

- [Configuración de la automatización de flujos de trabajo en Security Center](../../security-center/workflow-automation.md)

- [Configuración de respuestas automatizadas frente a amenazas en Azure Security Center](https://docs.microsoft.com/azure/security-center/tutorial-security-incident#triage-security-alerts)

- [Configuración de respuestas automatizadas frente a amenazas en Azure Sentinel](../../sentinel/tutorial-respond-threats-playbook.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="posture-and-vulnerability-management"></a>administración de posturas y vulnerabilidades

*Para más información, consulte [Azure Security Benchmark: administración de posturas y vulnerabilidades](/azure/security/benchmarks/security-controls-v2-vulnerability-management).*

### <a name="pv-1-establish-secure-configurations-for-azure-services"></a>PV-1: establecimiento de configuraciones seguras para servicios de Azure 

**Guía**: Las soluciones de administración de identidades y acceso de Microsoft ayudan a TI a proteger el acceso a aplicaciones y recursos en el entorno local y en la nube. Es importante que las organizaciones sigan los procedimientos recomendados de seguridad para asegurarse de que su implementación de administración de identidades y acceso sea segura y más resistente a los ataques. 

En función de la estrategia de implementación de administración de identidades y acceso, la organización debe seguir las instrucciones de procedimientos recomendados de Microsoft para proteger la infraestructura de identidad. 

Las organizaciones que colaboran con asociados externos también deben evaluar e implementar las configuraciones de gobernanza, seguridad y cumplimiento adecuadas para reducir el riesgo de seguridad y proteger los recursos confidenciales.

- [Procedimientos recomendados para la administración de identidades y la seguridad del control de acceso en Azure](../../security/fundamentals/identity-management-best-practices.md)

- [Cinco pasos para asegurar su infraestructura de identidad](../../security/fundamentals/steps-secure-identity.md)

- [Protección de la colaboración externa en Azure Active Directory y Microsoft 365](secure-external-access-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pv-2-sustain-secure-configurations-for-azure-services"></a>PV-2: sostenimiento de configuraciones seguras para servicios de Azure

**Guía**: La puntuación segura de Microsoft proporciona a las organizaciones una medida de su posición de seguridad y recomendaciones que pueden ayudar a proteger a las organizaciones frente a amenazas. Se recomienda que las organizaciones revisen periódicamente su puntuación de seguridad para ver las acciones de mejora sugeridas para mejorar su posición de seguridad de identidad. 

- [¿Qué es la puntuación segura de identidad en Azure Active Directory?](identity-secure-score.md)

- [Puntuación segura de Microsoft](/microsoft-365/security/mtp/microsoft-secure-score)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pv-8-conduct-regular-attack-simulation"></a>PV-8: realización de una simulaciones de ataques periódicas

**Guía**: Según sea necesario, realice pruebas de penetración o actividades de equipo rojo en los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos.
siga las reglas de compromiso de la prueba de penetración de Microsoft Cloud para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft.

- [Pruebas de penetración en Azure](../../security/fundamentals/pen-testing.md)

- [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

## <a name="governance-and-strategy"></a>Gobernanza y estrategia

*Para más información, consulte [Azure Security Benchmark: gobernanza y estrategia](/azure/security/benchmarks/security-controls-v2-governance-strategy).*

### <a name="gs-1-define-asset-management-and-data-protection-strategy"></a>GS-1: Definición de la estrategia de protección de datos y administración de recursos 

**Guía**: Asegúrese de documentar y comunicar una estrategia clara para la protección y supervisión continua de sistemas y datos. Dé prioridad a la detección, evaluación, protección y supervisión de los sistemas y datos críticos para la empresa. 
 

 
Esta estrategia debe incluir instrucciones, directivas y estándares documentados para los siguientes elementos: 
 
-   Norma de clasificación de datos de acuerdo con los riesgos empresariales
 
-   Visibilidad en la organización de seguridad de los riesgos y el inventario de recursos 
 
-   Aprobación de la organización de seguridad de los servicios de Azure para su uso 
 
-   Seguridad de los recursos durante su ciclo de vida
 
-   Estrategia de control de acceso necesaria según la clasificación de datos de la organización
 
-   Uso de las funcionalidades de protección de datos nativa de Azure y de terceros
 
-   Requisitos de cifrado de datos para casos de uso en tránsito y en reposo
 
-   Normas criptográficas adecuadas
 

 
Para más información, consulte las siguientes referencias: 
- [Recomendación para la arquitectura de seguridad de Azure: almacenamiento, datos y cifrado](/azure/architecture/framework/security/storage-data-encryption)
 

 
- [Aspectos básicos de la seguridad de Azure: seguridad, cifrado y almacenamiento de datos de Azure](../../security/fundamentals/encryption-overview.md) 

 
- [Cloud Adoption Framework: procedimientos recomendados de cifrado y seguridad de los datos de Azure](../../security/fundamentals/data-encryption-best-practices.md) 

 
- [Azure Security Benchmark: administración de recursos](/azure/security/benchmarks/security-benchmark-v2-asset-management) 

 
- [Azure Security Benchmark: protección de datos](/azure/security/benchmarks/security-benchmark-v2-data-protection)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="gs-2-define-enterprise-segmentation-strategy"></a>GS-2: Definición de una estrategia de segmentación empresarial 

**Guía**: Establezca en toda la empresa una estrategia para segmentar el acceso a los recursos mediante una combinación de identidad, red, aplicación, suscripción, grupo de administración y otros controles.

Equilibre concienzudamente la necesidad de separación de seguridad con la necesidad de habilitar el funcionamiento diario de los sistemas que necesitan comunicarse entre sí y acceder a los datos.

Asegúrese de que la estrategia de segmentación se implementa de forma coherente en todos los tipos de control, como la seguridad de red, los modelos de identidad y acceso, y los modelos de acceso y permisos de las aplicaciones, y los controles de los procesos humanos.

- [Guía de la estrategia de segmentación en Azure (vídeo)](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151)

- [Guía de la estrategia de segmentación en Azure (documento)](/security/compass/governance#enterprise-segmentation-strategy)

- [Alineación de la segmentación de red con la estrategia de segmentación empresarial](/security/compass/network-security-containment#align-network-segmentation-with-enterprise-segmentation-strategy)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="gs-3-define-security-posture-management-strategy"></a>GS-3: Definición de la estrategia de administración de la posición de seguridad

**Guía**: Mida y mitigue continuamente los riesgos de los recursos individuales y el entorno en el que se hospedan. Dé prioridad a los recursos de gran valor y a las superficies de ataque muy expuestas, como las aplicaciones publicadas, los puntos de entrada y salida de red, los puntos de conexión de usuario y administrador, etc.

- [Azure Security Benchmark: administración de posturas y vulnerabilidades](/azure/security/benchmarks/security-benchmark-v2-posture-vulnerability-management)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="gs-4-align-organization-roles-responsibilities-and-accountabilities"></a>GS-4: Alineación de los roles y responsabilidades de la organización

**Guía**: Asegúrese de documentar y comunicar una estrategia clara para los roles y las responsabilidades de la organización de seguridad. Dé prioridad a ofrecer una responsabilidad clara en las decisiones de seguridad, a proporcionar a todos los usuarios formación sobre el modelo de responsabilidad compartida y a los equipos técnicos sobre la tecnología para proteger la nube.

- [Procedimiento recomendado de seguridad de Azure 1. Personas: Formación del equipo sobre el recorrido de seguridad de la nube](/azure/cloud-adoption-framework/security/security-top-10#1-people-educate-teams-about-the-cloud-security-journey)

- [Procedimiento recomendado de seguridad de Azure 2. Personas: Formación del equipo en tecnología de seguridad de la nube](/azure/cloud-adoption-framework/security/security-top-10#2-people-educate-teams-on-cloud-security-technology)

- [Procedimiento recomendado de seguridad de Azure 3. Proceso: Asignación de responsabilidades por las decisiones de seguridad en la nube](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="gs-5-define-network-security-strategy"></a>GS-5: Definición de la estrategia de seguridad de red

**Guía**: Establezca un enfoque de seguridad de red de Azure como parte de la estrategia de control de acceso de seguridad general de su organización.  

Esta estrategia debe incluir instrucciones, directivas y estándares documentados para los siguientes elementos: 

-   Centralización de la responsabilidad de seguridad y la administración de redes

-   Modelo de segmentación de la red virtual alineado con la estrategia de segmentación empresarial

-   Estrategia de corrección en diferentes escenarios de amenazas y ataques

-   Estrategia de entrada y salida y perimetral de Internet

-   Estrategia de interconectividad entre el entorno local y la nube híbrida

-   Artefactos de seguridad de red actualizados (por ejemplo, diagramas de red y arquitectura de red de referencia)

Para más información, consulte las siguientes referencias:
- [Procedimiento recomendado de seguridad de Azure 11. Arquitectura: Establecimiento de una estrategia de seguridad unificada](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy)

- [Azure Security Benchmark: seguridad de red](/azure/security/benchmarks/security-benchmark-v2-network-security)

- [Azure Network Security Overview (Información general sobre Azure Network Security)](../../security/fundamentals/network-overview.md)

- [Estrategia para la arquitectura de red empresarial](/azure/cloud-adoption-framework/ready/enterprise-scale/architecture)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="gs-6-define-identity-and-privileged-access-strategy"></a>GS-6: Definición de la estrategia de acceso con privilegios e identidades

**Guía**: Establezca una identidad de Azure y enfoques de acceso con privilegios como parte de la estrategia de control de acceso de seguridad general de su organización.  

Esta estrategia debe incluir instrucciones, directivas y estándares documentados para los siguientes elementos: 

-   Un sistema de identidad y autenticación centralizado y su interconectividad con otros sistemas de identidad internos y externos

-   Métodos de autenticación sólida en diferentes casos de uso y condiciones

-   Protección de usuarios con privilegios elevados

-   Supervisión y control de anomalías en las actividades de los usuarios  

-   Proceso de revisión y conciliación del acceso y la identidad de los usuarios

Para más información, consulte las siguientes referencias:

- [Azure Security Benchmark: administración de identidades](/azure/security/benchmarks/security-benchmark-v2-identity-management)

- [Azure Security Benchmark: acceso con privilegios](/azure/security/benchmarks/security-benchmark-v2-privileged-access)

- [Procedimiento recomendado de seguridad de Azure 11. Arquitectura: Establecimiento de una estrategia de seguridad unificada](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy)

- [Información general sobre seguridad de administración de identidades de Azure](../../security/fundamentals/identity-management-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="gs-7-define-logging-and-threat-response-strategy"></a>GS-7: Definición de la estrategia de registro y respuesta a amenazas

**Guía**: Establezca una estrategia de registro y respuesta a amenazas para detectar y corregir rápidamente las amenazas mientras cumple los requisitos de cumplimiento. Para dar prioridad, proporcione a los analistas alertas de alta calidad y experiencias sin problemas para que puedan centrarse en las amenazas en lugar de los pasos manuales y de integración. 

Esta estrategia debe incluir instrucciones, directivas y estándares documentados para los siguientes elementos: 

-   Las responsabilidades y el rol de la organización en las operaciones de seguridad (SecOps) 

-   Un proceso de respuesta a incidentes bien definido que esté alineado con NIST u otro marco del sector. 

-   Captura y retención de registros para admitir la detección de amenazas, la respuesta ante incidentes y las necesidades de cumplimiento

-   Visibilidad y correlación centralizada de la información sobre amenazas, mediante SIEM, funcionalidades nativas de Azure y otros orígenes 

-   Plan de comunicación y notificación con sus clientes, proveedores y entidades públicas de interés

-   Uso de plataformas nativas de Azure y de terceros para el tratamiento de incidentes, como el registro y la detección de amenazas, los análisis forenses y la corrección y erradicación de ataques

-   Procesos para controlar incidentes y actividades posteriores a incidentes, como las lecciones aprendidas y la retención de pruebas

Para más información, consulte las siguientes referencias:

- [Azure Security Benchmark: registro y detección de amenazas](/azure/security/benchmarks/security-benchmark-v2-logging-threat-detection)

- [Azure Security Benchmark: respuesta a incidentes](/azure/security/benchmarks/security-benchmark-v2-incident-response)

- [Procedimiento recomendado de seguridad de Azure 4. Proceso: Actualización de los procesos de respuesta a incidentes para la nube](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

- [Guía de decisiones sobre registros e informes en Azure Adoption Framework](/azure/cloud-adoption-framework/decision-guides/logging-and-reporting/)

- [Escala, administración y supervisión empresarial de Azure](/azure/cloud-adoption-framework/ready/enterprise-scale/management-and-monitoring)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [Información general sobre Azure Security Benchmark V2](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
