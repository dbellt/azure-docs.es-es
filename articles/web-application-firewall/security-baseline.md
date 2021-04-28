---
title: Línea base de seguridad de Azure Web Application Firewall
description: La base de referencia de seguridad de Azure Web Azure Web Application Firewall proporciona instrucciones de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: web-application-firewall
ms.topic: conceptual
ms.date: 04/08/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 6aa3e2b84c4349e2134ddeb2a60fd193f56f84e8
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107875976"
---
# <a name="azure-security-baseline-for-azure-web-application-firewall"></a>Línea base de seguridad de Azure Web Application Firewall

Esta línea de base de seguridad aplica la guía de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview-v1.md) a Azure Web Application Firewall. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa por los **controles de seguridad** definidos por Azure Security Benchmark y las directrices aplicables a Azure Web Application Firewall. 

> [!NOTE]
> Se han excluido los **controles** que no son aplicables a Azure Web Application Firewall o cuya responsabilidad es de Microsoft. Para ver cómo Azure Web Application Firewall se asigna por completo a Azure Security Benchmark, consulte el **[archivo de asignación de base de referencia de seguridad de Azure Web Application Firewall completo](https://github.com/MicrosoftDocs/SecurityBenchmarks/raw/master/Azure%20Offer%20Security%20Baselines/1.1/azure-web-application-firewall-security-baseline-v1.1.xlsx)** .

## <a name="network-security"></a>Seguridad de redes

*Para más información, consulte [Azure Security Benchmark: seguridad de red](../security/benchmarks/security-control-network-security.md).*

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: Utilice Microsoft Azure Web Application Firewall (WAF) para una protección centralizada de las aplicaciones web frente a vulnerabilidades de seguridad comunes, como la inyección de SQL y el scripting entre sitios. 

- Modo de detección: Utilice este modo para entender el tráfico de red y comprender y revisar falsos positivos. Supervisa y registra todas las alertas de amenazas. Asegúrese de que el registro de diagnósticos y WAF está seleccionado y activado. Tenga en cuenta que el WAF no bloquea solicitudes entrantes cuando se ejecuta en modo de detección.
- Modo de prevención: Bloquea las intrusiones y los ataques que detectan las reglas. Un atacante recibe una excepción de "403: acceso no autorizado" y se cierra la conexión. El modo de prevención registra dichos ataques en los registros del WAF.

Línea base del tráfico de red con el modo de detección de WAF. Después de determinar las necesidades de tráfico, ponga el WAF en modo de prevención para bloquear el tráfico no deseado.

Haga un seguimiento de las recomendaciones de gravedad alta de Security Center para los recursos habilitados para web que no estén protegidos por WAF.  

- [Reglas y grupos de reglas de CRS de Firewall de aplicaciones Web](ag/application-gateway-crs-rulegroups-rules.md) 

- [Modos de WAF en Application Gateway](https://docs.microsoft.com/azure/web-application-firewall/ag/ag-overview#waf-modes)

- [Modos WAF en Front Door](https://docs.microsoft.com/azure/web-application-firewall/afds/afds-overview#waf-modes)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Denegación de las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: Utilice reglas personalizadas con Azure Web Application Firewall (WAF) para permitir y bloquear el tráfico. Por ejemplo, puede bloquear todo el tráfico procedente de un intervalo de direcciones IP. Configure Azure WAF para que se ejecute en el modo de prevención, lo cual bloquea las intrusiones y los ataques que detectan las reglas. Un atacante recibe una excepción de "403: acceso no autorizado" y se cierra la conexión. El modo de prevención registra dichos ataques en los registros del WAF.

- [Modos de WAF en Application Gateway](https://docs.microsoft.com/azure/web-application-firewall/ag/ag-overview#waf-modes)

- [Modos WAF en Front Door](https://docs.microsoft.com/azure/web-application-firewall/afds/afds-overview#waf-modes)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Network**:

[!INCLUDE [Resource Policy for Microsoft.Network 1.4](../../includes/policy/standards/asb/rp-controls/microsoft.network-1-4.md)]

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Guía**: Azure Web Application Firewall (WAF) es el componente principal de las protecciones de aplicaciones web de Azure. Utilice Azure WAF para proporcionar una protección centralizada a las aplicaciones web a partir de vulnerabilidades de seguridad y vulnerabilidades comunes con el conjunto de reglas administradas preconfiguradas contra firmas de ataque conocidas de las 10 categorías principales de OWASP.

Personalice las reglas y grupos de reglas de Azure WAF para satisfacer los requisitos de la aplicación web y eliminar los falsos positivos. Asocie una directiva de Azure WAF a cada sitio detrás de la WAF para permitir la configuración específica del sitio. Azure WAF admite el filtrado geográfico, la limitación de velocidad y las reglas de conjunto de reglas predeterminadas administradas por Azure. y pueden crearse reglas personalizadas para satisfacer las necesidades de una aplicación. 

Configure Azure WAF para que se ejecute en el modo de prevención después de establecer una línea de tiempo en el modo de detección del tráfico de red durante un período de tiempo determinado. Azure WAF bloquea cual bloquea las intrusiones y los ataques que detectan las reglas en el modo de prevención. Un atacante recibe una excepción de "403: acceso no autorizado" y se cierra la conexión. El modo de prevención registra dichos ataques en los registros del WAF.

- [Modos de WAF en Application Gateway](https://docs.microsoft.com/azure/web-application-firewall/ag/ag-overview#waf-modes)

- [Modos WAF en Front Door](https://docs.microsoft.com/azure/web-application-firewall/afds/afds-overview#waf-modes)

- [Reglas y grupos de reglas de CRS de firewall de aplicaciones Web](https://docs.microsoft.com/azure/web-application-firewall/ag/application-gateway-crs-rulegroups-rules?tabs=owasp31)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: Cree, asocie y organice lógicamente los recursos de una suscripción de Azure con etiquetas para la detección de configuraciones de aplicaciones comunes (por ejemplo, Apache e IIS). 

Aplique reglas y grupos de reglas a las directivas de Azure Web Application Firewall (WAF) basadas en los metadatos de etiqueta aplicados.

- [Directivas de WAF sobre Application Gateway](/cli/azure/network/application-gateway/waf-policy)

- [Directivas de WAF sobre Front Door](/cli/azure/network/front-door/waf-policy)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: Utilice etiquetas para los grupos de seguridad de red asociados con Azure Web Application Firewall (WAF) en la subred de Azure Application Gateway, así como cualquier otro recurso relacionado con la seguridad de red y el flujo de tráfico. En el caso de reglas de grupo de seguridad de red individuales, utilice el campo "Descripción" para especificar la necesidad del negocio, la duración y otra información de cualquier regla que permita el tráfico hacia o desde una red.

Use cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor", para asegurarse de que todos los recursos se creen con etiquetas y para notificarle los recursos no etiquetados existentes.

Seleccione Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

- [Creación de una red virtual](../virtual-network/quick-create-portal.md)

- [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: Utilice el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en las opciones y recursos de red relacionados con las implementaciones de Azure Web Application Firewall (WAF). Cree alertas en Azure Monitor que se desencadenen cuando se produzcan cambios en la configuraciones o recursos de red críticos.

- [Visualización y recuperación de eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/essentials/activity-log#view-the-activity-log)

- [Creación de alertas en Azure Monitor](../azure-monitor/alerts/alerts-activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Azure Security Benchmark: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: Cree una regla de red para Azure Web Application Firewall (WAF) para permitir el acceso a un servidor NTP con el puerto y el protocolo adecuados, como el puerto 123 a través de UDP.

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Configure los registros de Azure Web Application Firewall (WAF) para enviarlos a una solución central de administración de registros de seguridad, como Azure Sentinel, o a un SIEM de terceros. Estos registros incluyen los registros de Azure Activity, Diagnostic y WAF en tiempo real. Dichos registros pueden verse en distintas herramientas, como Azure Monitor, Excel y Power BI. Los registros de Azure Web Application Firewall proporcionan información sobre los datos que Azure WAF evalúa, empareja y bloquea.

Azure Sentinel tiene un libro de Azure WAF integrado que proporciona información general acerca de los eventos de seguridad de Azure WAF. Este libro incluye eventos, reglas coincidentes y bloqueadas, y todo lo demás que se registra en los registros del firewall. Esta telemetría puede utilizarse para iniciar la automatización de la guía para notificar o tomar medidas de corrección basadas en eventos de WAF recopilados por Azure Sentinel.

- [Visualización de registro de actividad](../azure-resource-manager/management/view-activity-logs.md)

- [Registros de diagnóstico para Application Gateway](../application-gateway/application-gateway-diagnostics.md)

- [Conectar datos del firewall de aplicaciones web de Microsoft a Azure Sentinel](../sentinel/connect-azure-waf.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Habilite el inicio de sesión en los recursos de Azure Web Application Firewall (WAF) para acceder a los registros de auditoría, seguridad y diagnóstico. Azure Web Application Firewall proporciona informes detallados sobre cada una de las amenazas detectadas que están disponibles en los registros de diagnóstico configurados. Los registros de actividad, que están disponibles automáticamente, incluyen el origen del evento, la fecha, el usuario, la marca de tiempo, las direcciones de origen y de destino, y otros elementos útiles.

- [Información general acerca del registro](https://docs.microsoft.com/azure/web-application-firewall/ag/ag-overview#logging)

- [Información general acerca de las consultas de registro de Azure Monitor](../azure-monitor/logs/log-query-overview.md)

- [Información general acerca de los registros de plataforma de Azure](../azure-monitor/essentials/platform-logs-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: Envíe los registros de Azure Web Application Firewall (WAF) a una cuenta de almacenamiento personalizada y defina las directivas de retención. Utilice Azure Monitor para establecer el período de retención del área de trabajo de Log Analytics en función de los requisitos de cumplimiento de su organización.
- [la supervisión para una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/manage-storage-analytics-logs#configure-logging)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Guía**: Azure Web Application Firewall (WAF) proporciona informes detallados sobre cada amenaza detectada. Los registros están integrados con los registros de Azure Diagnostics, los cuales proporcionan información valiosa acerca de las operaciones y los errores que son importantes para la auditoría, así como para solucionar problemas. 

Las instancias de Azure WAF están integradas con Security Center para enviar alertas e información de estado para los informes. Utilice las recomendaciones de Security Center para detectar aplicaciones web no protegidas y proteger los recursos vulnerables. 

Azure Sentinel tiene un libro de WAF y eventos de firewall integrado que proporciona información general acerca de los eventos de seguridad de WAF. Estos incluyen eventos, reglas coincidentes y bloqueadas, y todo lo demás que se registra en los registros del firewall.

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](/azure/azure-monitor/platform/activity-log)

- [Habilitación de la configuración de diagnóstico de Azure Application Gateway](../application-gateway/application-gateway-diagnostics.md)

- [Supervisión de métricas y registro en Azure Front Door](../frontdoor/front-door-diagnostics.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Habilite la configuración de diagnóstico del registro de actividad de Azure, así como la configuración de diagnóstico de Azure WAF, y envíe los registros a un área de trabajo de Log Analytics. Realice consultas en Log Analytics para buscar términos, identificar tendencias, analizar patrones y proporcionar otra información detallada basada en los datos recopilados. Cree alertas para actividades anómalas basadas en métricas de WAF. Por ejemplo, si el número de solicitudes bloqueadas supera "X" o "Y".

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](/azure/azure-monitor/essentials/diagnostic-settings-legacy)

- [Creación de alertas en Azure](../azure-monitor/alerts/tutorial-response.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: Implemente el firewall de aplicaciones web (WAF) de Azure delante de las aplicaciones web críticas para la inspección adicional del tráfico entrante. 

Azure WAF proporciona una protección centralizada de las aplicaciones web frente a vulnerabilidades comunes y otras vulnerabilidades y protege las aplicaciones mediante una inspección del tráfico web entrante para bloquear ataques como inyecciones de SQL, scripts entre sitios, cargas de malware y ataques de DDoS.

- [Implementación de WAF de Azure](ag/create-waf-policy-ag.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para más información, consulte [Azure Security Benchmark: identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: Azure Active Directory (Azure AD) tiene roles integrados que son consultables y deben asignarse explícitamente. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc para detectar cuentas que son miembros de grupos administrativos.

- [Obtención de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0&amp;preserve-view=true)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0&amp;preserve-view=true)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: No hay ninguna asignación de administrador local disponible dentro de WAF. Sin embargo, podría haber roles de administrador de Azure Active Directory (Azure AD) en el entorno que podrían permitir el control de la administración de los recursos de Azure WAF.
Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas que tengan acceso a las instancias de Azure Web Application Firewall (WAF). Use la identidad de Security Center y las características de administración de acceso para supervisar el número de cuentas administrativas.

- [Descripción de identidad y acceso en Azure Security Center](../security-center/security-center-identity-access.md)

- [Descripción sobre cómo crear usuarios administradores en Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/howto-create-users#the-server-admin-account)

- [Uso de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: habilite la autenticación multifactor de Azure Active Directory (Azure AD) y siga las recomendaciones de administración de identidades y acceso de Security Center.

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: Utilice una estación de trabajo de acceso con privilegios (PAW) con la autenticación multifactor configurada para iniciar sesión en recursos de Azure Web Application Firewall (WAF) y recursos relacionados y configurarlos.

- [Más información sobre las estaciones de trabajo con privilegios de acceso](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: Use los informes de seguridad de Azure Active Directory (Azure AD) para la generación de registros y alertas cuando se produzcan actividades sospechosas o no seguras en el entorno. Utilice Security Center para supervisar la actividad de identidad y acceso.

- [Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo](../active-directory/identity-protection/overview-identity-protection.md)

- [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Instrucciones**: Configure la condición de ubicación de una directiva de acceso condicional y administre las ubicaciones con nombre.

Cree agrupaciones lógicas de rangos de direcciones IP o de países y regiones con ubicación con nombre. Restrinja el acceso a recursos confidenciales, como los secretos de Azure Key Vault, a las ubicaciones con nombre configuradas.

- [¿Qué es la condición de ubicación del acceso condicional de Azure Active Directory (Azure AD)?](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización. Azure AD protege los datos mediante el cifrado seguro para datos en reposo y en tránsito, y también sales, hashes y almacena de forma segura las credenciales de usuario.
- [Procedimiento para crear y configurar una instancia de Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Instrucciones**: Azure Active Directory (Azure AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. Revise el acceso de los usuarios con regularidad para asegurarse de que solo los usuarios activos tengan el acceso continuado.

- [Descripción de los informes de Azure AD](/azure/active-directory/reports-monitoring)

- [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: Integre la actividad de inicio de sesión de Azure Active Directory (Azure AD) y los orígenes del registro de eventos de riesgo y auditoría, con cualquier herramienta de SIEM o supervisión, como Azure Sentinel.

Para simplificar este proceso, cree una configuración de diagnóstico para las cuentas de usuario de Azure AD, y envíe los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Configure las alertas deseadas en el área de trabajo de Log Analytics.

- [Integración de los registros de actividad de Azure en Azure Monitor](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12: Alerta de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Use las características de protección de identidad y detección de riesgo de Azure Active Directory (Azure AD) para configurar respuestas automatizadas a las acciones sospechosas que se detecten relacionadas con las identidades de los usuarios. Ingiera datos en Azure Sentinel para investigarlos más.

- [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-protection"></a>Protección de datos

*Para más información, consulte [Azure Security Benchmark: protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: Utilice etiquetas para ayudar a realizar el seguimiento de los recursos de Azure Web Application Firewall (WAF) y recursos relacionados que almacenan o procesan información confidencial.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Guía**: Implemente también el aislamiento mediante suscripciones independientes y grupos de administración para dominios de seguridad individuales, como el tipo de entorno y el nivel de confidencialidad de los datos, por ejemplo, entornos de desarrollo, prueba y producción. 

Controle el acceso a los recursos de Azure con el acceso basado en roles de Azure Active Directory (Azure AD) (Azure RBAC).

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Instrucciones**: Cifre toda la información confidencial en tránsito. Asegúrese de que los clientes que se conectan a los recursos de las instancias de Azure Web Application Firewall (WAF) y recursos relacionados pueden negociar TLS 1.2 o superior.

Siga las recomendaciones de Security Center para el cifrado en reposo y el cifrado en tránsito, si procede.

- [Descripción del cifrado en tránsito con Azure](https://docs.microsoft.com/azure/security/fundamentals/encryption-overview#encryption-of-data-in-transit)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4.6: Uso de RBAC de Azure para administrar el acceso a los recursos 

**Guía**: Controle el acceso a los recursos de Azure como Web Application Firewall con el control de acceso basado en rol de Azure (RBAC de Azure).

- [Configuración de RBAC en Azure](../role-based-access-control/role-assignments-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Utilice el cifrado en reposo para todos los recursos de Azure, incluidos los de Azure Web Application Firewall (WAF) y otros recursos relacionados. Microsoft recomienda permitir que Azure administre las claves de cifrado, pero existe la opción de administrar sus propias claves en algunas instancias.

- [Descripción del cifrado en reposo en Azure](../security/fundamentals/encryption-atrest.md)

- [Configuración de las claves de cifrado administradas por el cliente](https://docs.microsoft.com/azure/storage/common/customer-managed-keys-configure-key-vault?tabs=portal)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Configure Azure Web Application Firewall (WAF) para que se ejecute en el modo de prevención después de establecer una línea de tiempo en el modo de detección del tráfico de red durante un período de tiempo predeterminado. 

En el modo de prevención, Azure WAF bloquea cual bloquea las intrusiones y los ataques que detectan las reglas. El atacante recibe una excepción de "403: acceso no autorizado" y se cierra la conexión. El modo de prevención registra dichos ataques en los registros del WAF.

- [Introducción a la integración entre Application Gateway y Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-partner-integration#overview)

- [Modos de WAF en Application Gateway](https://docs.microsoft.com/azure/web-application-firewall/ag/ag-overview#waf-modes)

- [Modos WAF en Front Door](https://docs.microsoft.com/azure/web-application-firewall/afds/afds-overview#waf-modes)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para más información, consulte [Azure Security Benchmark: inventario y administración de recursos](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Utilice Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, puertos y protocolos, etc.) dentro de las suscripciones.

Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y enumere todas las suscripciones de Azure, así como los recursos de las suscripciones. Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

- [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Visualización de las suscripciones de Azure](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-4.8.0&amp;preserve-view=true)

- [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: Utilice etiquetas en las directivas de Azure Web Application Firewall (WAF) Las etiquetas pueden asociarse a recursos y aplicarse de manera lógica para organizar el acceso a estos recursos en una suscripción de cliente. 

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: Utilice el etiquetado, los grupos de administración y las suscripciones independientes, según corresponda, para organizar y hacer un seguimiento de los recursos relacionados con Azure WAF. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: Cree un inventario de los recursos aprobados, incluida la configuración según las necesidades de su organización.

use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones. Use Azure Resource Graph para consultar y detectar recursos dentro de sus suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en sus suscripciones.
Utilice Azure Resource Graph para consultar o detectar recursos de Azure Web Application Firewall (WAF) dentro de sus suscripciones. Asegúrese de que todos los recursos de Azure WAF y recursos relacionados presentes en el entorno estén aprobados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: Supervise y quite los recursos de Azure WAF no aprobados con Azure Policy para denegar la implementación de Azure WAF o un tipo determinado de WAF; por ejemplo, Azure WAF v1 frente a V2.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: Use Azure Policy para restringir qué servicios puede aprovisionar en su entorno.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](/azure/governance/policy/samples/built-in-policies#general)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Guía**: Utilice el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure".

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: Azure Web Application Firewall (WAF) puede asociarse a diferentes entornos a través de redes, grupos de recursos o suscripciones para separar las aplicaciones de alto riesgo.

- [Información general sobre Azure Virtual Network](../virtual-network/virtual-networks-overview.md)

- [Organización de los recursos con grupos de administración de Azure](../governance/management-groups/overview.md)

- [Guía de decisiones de suscripción](/azure/cloud-adoption-framework/decision-guides/subscriptions/)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Azure Security Benchmark: configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Defina e implemente configuraciones de seguridad estándar para la configuración de red relacionada con las implementaciones de Azure Web Application Firewall (WAF).

Utilice alias de Azure Policy en el espacio de nombres "Microsoft.Network" para crear directivas personalizadas que permitan auditar o aplicar la configuración de red de Azure Application Gateways, Virtual Networks, grupos de seguridad de red y definiciones de directivas integradas.

- [Visualización de los alias de Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-4.8.0&amp;preserve-view=true)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Utilice los efectos de las directivas [deny] y [deploy if not exist] de Azure Policy para aplicar una configuración segura en los recursos de Azure Web Application Firewall (WAF) y recursos relacionados. 

Utilice plantillas de Azure Resource Manager para mantener la configuración de seguridad de los recursos de Azure WAF y recursos relacionados que requiere su organización.

- [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

- [Creación y administración de directivas para aplicar el cumplimiento](../governance/policy/tutorials/create-and-manage.md)

- [Información general sobre plantillas de Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Utilice Azure DevOps para almacenar y administrar de forma segura el código, como directivas personalizadas de Azure y plantillas de Azure Resource Manager.

Conceda o deniegue permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integra con Azure DevOps, o en Azure AD si se integra con Team Foundation Server (TFS).

- [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops&amp;preserve-view=true)

- [Acerca de los permisos y los grupos en Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: use las definiciones integradas en Azure Policy junto con los alias de esta misma aplicación en el espacio de nombres "Microsoft.Network" para crear directivas personalizadas que permitan auditar y aplicar las configuraciones, así como enviar alertas sobre ellas. Desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Documentación de Azure Policy](/azure/governance/policy)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: use las definiciones integradas en Azure Policy junto con los alias de esta misma aplicación en el espacio de nombres "Microsoft.Network" para crear directivas personalizadas que permitan auditar y aplicar las configuraciones, así como enviar alertas sobre ellas. 

Utilice los efectos de la directiva de Azure Policy [audit], [deny] y [deploy if not exist] para aplicar automáticamente las configuraciones en los recursos de Azure.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Documentación de Azure Policy](/azure/governance/policy)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: Use Azure Key Vault para almacenar certificados de forma segura. Azure Key Vault es un almacén de secretos administrado por la plataforma que puede usar para proteger los secretos, las claves y los certificados SSL. 

Azure Application Gateway admite la integración con Key Vault para certificados de servidor adjuntos a clientes de escucha con HTTPS habilitado. 

- [Configuración de la terminación SSL con certificados de Key Vault mediante Azure PowerShell](../application-gateway/configure-keyvault-ps.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Guía**: Implemente el escáner de credenciales para identificar las credenciales del código, lo cual también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.
- [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Azure Security Benchmark: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Instrucciones**: Asegúrese de que la eliminación temporal está habilitada para Azure Key Vault. La eliminación temporal permite la recuperación de almacenes de claves y objetos de almacén eliminados, como claves, secretos y certificados.

- [Uso de la eliminación temporal de Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/key-vault-recovery?tabs=azure-powershell&amp;preserve-view=true)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Azure Security Benchmark: respuesta ante incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Guía**: desarrolle una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia. 

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/) 

- [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/) 

- [Uso de la guía de control de incidentes de seguridad de equipos de NIST para la creación de su propio plan de respuesta a incidentes](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Guía**: Security Center asigna una gravedad a cada alerta para ayudar a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en la métrica utilizados para emitir la alerta, así como en el nivel de confianza de que se produjo un intento malintencionado detrás de la actividad que provocó la alerta.

Marque las suscripciones de forma clara (por ejemplo, producción o no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.
- [Lea la publicación de NIST: guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.
- [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Guía**: exporte sus alertas y recomendaciones de Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Utilice el conector de datos de Azure Security Center para enviar las alertas a Azure Sentinel, de acuerdo con los requisitos de la organización.

- [Configuración de la exportación continua](../security-center/continuous-export.md)

- [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: Utilice la característica de automatización del flujo de trabajo de Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad.
- [Configuración de la automatización de flujo de trabajo y Logic Apps](../security-center/workflow-automation.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Azure Security Benchmark: Pruebas de penetración y ejercicios del equipo rojo](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de pruebas de penetración de sitios activos y ataques simulados en la infraestructura en la nube, los servicios y las aplicaciones administrados por Microsoft. 

- [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [Información general sobre Azure Security Benchmark V2](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
