---
title: Línea base de seguridad de Azure para Cognitive Services
description: La línea base de seguridad de Cognitive Services proporciona orientaciones sobre procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 03/30/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: a7f9570a68102eb20c84942ddf9ac73de033f01c
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106076605"
---
# <a name="azure-security-baseline-for-cognitive-services"></a>Línea base de seguridad de Azure para Cognitive Services

Esta línea de base de seguridad aplica la guía de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview-v1.md) en Cognitive Services. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa mediante los **controles de seguridad** definidos en Azure Security Benchmark y las directrices relacionadas aplicables a Cognitive Services. Se han excluido los **controles** que no son aplicables a Cognitive Services o cuya responsabilidad es de Microsoft.

Para ver cómo Cognitive Services se asigna por completo a Azure Security Benchmark, consulte el [archivo completo de asignación de la línea base de seguridad de Cognitive Services](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Seguridad de redes

*Para más información, consulte [Azure Security Benchmark: seguridad de red](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: Microsoft Azure Cognitive Services proporciona un modelo de seguridad por niveles. Este modelo le permite proteger las cuentas de Cognitive Services en un subconjunto específico de redes. Cuando se configuran las reglas de red, solo las aplicaciones que solicitan datos del conjunto especificado de redes pueden acceder a la cuenta. Puede limitar el acceso a los recursos con el filtrado de solicitudes, lo que solo permite solicitudes que se originan a partir de direcciones IP especificadas, intervalos IP o de una lista de subredes de instancias de Azure Virtual Network.

La compatibilidad de la red virtual y el punto de conexión de servicio con Cognitive Services se limita a un conjunto específico de regiones.

- [Configuración de redes virtuales de Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-virtual-networks?tabs=portal)

- [Información general sobre las instancias de Azure Virtual Network](../virtual-network/virtual-networks-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes e interfaces de red

**Guía**: cuando se implementan máquinas virtuales en la misma red virtual que el contenedor de Cognitive Services, puede utilizar los grupos de seguridad de red para reducir el riesgo de filtración de datos. Habilite los registros de flujo de los grupos de seguridad de red y envíelos a una cuenta de Azure Storage para auditar el tráfico. También puede enviar registros de flujo de los grupos de seguridad de red a un área de trabajo de Log Analytics y usar Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

- [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: si usa Cognitive Services en un contenedor, puede aumentar la implementación del contenedor con una solución de firewall de aplicación web frontal que filtre el tráfico malintencionado y admita el cifrado TLS de un extremo a otro, lo que le permitirá garantizar la privacidad y seguridad del punto de conexión del contenedor. 

Tenga en cuenta que los contenedores de Cognitive Services son necesarios para enviar información de medición con fines de facturación. La única excepción son los contenedores sin conexión, ya que siguen otra metodología de facturación. Si no se permite la enumeración de varios canales de red en los que se basan los contenedores de Cognitive Services, el contenedor no funcionará. El host debe permitir que el puerto 443 y los siguientes dominios aparezcan:

- *.cognitive.microsoft.com
- *.cognitiveservices.azure.com

Tenga en cuenta también que debe deshabilitar la inspección profunda de paquetes para la solución de firewall en los canales seguros que los contenedores de Cognitive Services crean en los servidores de Microsoft. Si no lo hace, el contenedor no funcionará correctamente.

- [Descripción de la seguridad de los contenedores de Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support#azure-cognitive-services-container-security)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Denegación de las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: al implementar máquinas virtuales en la misma red virtual que el contenedor de Cognitive Services, defina e implemente las configuraciones de seguridad estándar para los recursos de red relacionados con Azure Policy. Use alias de Azure Policy en los espacios de nombres "Microsoft.CognitiveServices" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las instancias de Azure Cache for Redis. También puede utilizar definiciones de directivas integradas como:

- Se debe habilitar DDoS Protection estándar

Use Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como las plantillas de Azure Resource Manager, el control de acceso basado en roles de Azure (RBAC de Azure) y las directivas, en una única definición de un plano técnico. Aplique fácilmente el plano técnico a nuevas suscripciones y entornos, y ajuste el control y la administración mediante el control de versiones.

si usa Cognitive Services en un contenedor, puede aumentar la implementación del contenedor con una solución de firewall de aplicación web frontal que filtre el tráfico malintencionado y admita el cifrado TLS de un extremo a otro, lo que le permitirá garantizar la privacidad y seguridad del punto de conexión del contenedor. 

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)

- [Descripción de la seguridad de los contenedores de Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support#azure-cognitive-services-container-security)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Guía**: cuando se implementan máquinas virtuales en la misma red virtual que el contenedor de Cognitive Services, puede utilizar los grupos de seguridad de red para reducir el riesgo de filtración de datos. Habilite los registros de flujo de los grupos de seguridad de red y envíelos a una cuenta de Azure Storage para auditar el tráfico. También puede enviar registros de flujo de los grupos de seguridad de red a un área de trabajo de Log Analytics y usar Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

- [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implementación de sistemas de prevención de intrusiones y de detección de intrusiones (IDS/IPS) basados en la red.

**Instrucciones**: si usa Cognitive Services en un contenedor, puede aumentar la implementación del contenedor con una solución de firewall de aplicación web frontal que filtre el tráfico malintencionado y admita el cifrado TLS de un extremo a otro, lo que le permitirá garantizar la privacidad y seguridad del punto de conexión del contenedor.  Seleccione una oferta de Azure Marketplace que admita la funcionalidad de IDS/IPS con la capacidad de deshabilitar la inspección de carga.

Tenga en cuenta que los contenedores de Cognitive Services son necesarios para enviar información de medición con fines de facturación. La única excepción son los contenedores sin conexión, ya que siguen otra metodología de facturación. Si no se permite la enumeración de varios canales de red en los que se basan los contenedores de Cognitive Services, el contenedor no funcionará. El host debe permitir que el puerto 443 y los siguientes dominios aparezcan:

- *.cognitive.microsoft.com
- *.cognitiveservices.azure.com

Tenga en cuenta también que debe deshabilitar la inspección profunda de paquetes para la solución de firewall en los canales seguros que los contenedores de Cognitive Services crean en los servidores de Microsoft. Si no lo hace, el contenedor no funcionará correctamente.

- [Descripción de la seguridad de los contenedores de Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support#azure-cognitive-services-container-security)

- [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Guía**: si usa Cognitive Services en un contenedor, puede aumentar la implementación del contenedor con una solución de firewall de aplicación web frontal que filtre el tráfico malintencionado y admita el cifrado TLS de un extremo a otro, lo que le permitirá garantizar la privacidad y seguridad del punto de conexión del contenedor. 

Tenga en cuenta que los contenedores de Cognitive Services son necesarios para enviar información de medición con fines de facturación. La única excepción son los contenedores sin conexión, ya que siguen otra metodología de facturación. Si no se permite la enumeración de varios canales de red en los que se basan los contenedores de Cognitive Services, el contenedor no funcionará. El host debe permitir que el puerto 443 y los siguientes dominios aparezcan:

- *.cognitive.microsoft.com
- *.cognitiveservices.azure.com

Tenga en cuenta también que debe deshabilitar la inspección profunda de paquetes para la solución de firewall en los canales seguros que los contenedores de Cognitive Services crean en los servidores de Microsoft. Si no lo hace, el contenedor no funcionará correctamente.

- [Descripción de la seguridad de los contenedores de Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-container-support#azure-cognitive-services-container-security)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: Puede usar etiquetas de servicio de red virtual para definir controles de acceso a la red en los grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, ApiManagement) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico del servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

También puede usar grupos de seguridad de aplicaciones para simplificar una configuración de seguridad compleja. Los grupos de seguridad de aplicaciones le permiten configurar la seguridad de red como una extensión natural de la estructura de una aplicación, lo que le permite agrupar máquinas virtuales y directivas de seguridad de red basadas en esos grupos.

- [Etiquetas de servicio de red virtual](../virtual-network/service-tags-overview.md)

- [Grupos de seguridad de aplicaciones](https://docs.microsoft.com/azure/virtual-network/network-security-groups-overview#application-security-groups)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: defina e implemente configuraciones de seguridad estándar para los recursos de red relacionados con el contenedor de Cognitive Services mediante Azure Policy. Use alias de Azure Policy en los espacios de nombres "Microsoft.CognitiveServices" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las instancias de Azure Cache for Redis.

También puede utilizar Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como las plantillas de Azure Resource Manager, el control de acceso basado en roles (RBAC de Azure) y las directivas, en una única definición de un plano técnico. Aplique fácilmente el plano técnico a nuevas suscripciones y entornos, y ajuste el control y la administración mediante el control de versiones.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Instrucciones**: use etiquetas para los recursos de red asociados a la implementación del contenedor de Cognitive Services a fin de organizarlos lógicamente en una taxonomía.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en los recursos de red relacionados con el contenedor de Cognitive Services. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

- [Visualización y recuperación de eventos del registro de actividad de Azure](/azure/azure-monitor/platform/activity-log#view-the-activity-log)

- [Creación de alertas en Azure Monitor](/azure/azure-monitor/platform/alerts-activity-log)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Azure Security Benchmark: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para su archivo. Los registros de actividad proporcionan información sobre las operaciones llevadas a cabo en el contenedor de Cognitive Services en el nivel del plano de control. Con los datos del registro de actividad de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en el nivel del plano de control para las instancias de Azure Cache for Redis.

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](/azure/azure-monitor/platform/activity-log)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Para el registro de la auditoria del plano de control, habilite la configuración de diagnóstico del registro de actividad de Azure y envíela a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para su archivo. Con los datos del registro de actividad de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en el nivel del plano de control para los recursos de Azure.

Además, Cognitive Services envía eventos de diagnóstico que se pueden recopilar y usar para elaborar análisis, alertas e informes. Puede usar Azure Portal para configurar los valores de diagnóstico de un contenedor de Cognitive Services. Puede enviar uno o varios eventos de diagnóstico a una cuenta de almacenamiento, un centro de eventos o un área de trabajo de Log Analytics.

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](/azure/azure-monitor/platform/diagnostic-settings-legacy)

- [Uso de la configuración de diagnóstico en Azure Cognitive Services](diagnostic-logging.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

- [Configuración de parámetros de retención de registros de áreas de trabajo de Log Analytics](/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Guía**: Habilite la configuración de diagnóstico en el registro de actividad de Azure y envíe los registros a un área de trabajo de Log Analytics. Estos registros proporcionan datos exhaustivos y frecuentes sobre el funcionamiento de un recurso que se utilizan para la identificación y depuración del problema. Realice consultas en Log Analytics para buscar términos, identificar tendencias, analizar patrones y proporcionar muchas otras conclusiones basadas en los datos del registro de actividad que se pueden recopilar para Azure Cognitive Services.

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](/azure/azure-monitor/platform/activity-log)

- [Recopilación y análisis de registros de actividad de Azure en un área de trabajo de Log Analytics en Azure Monitor](/azure/azure-monitor/platform/activity-log)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: puede generar alertas sobre las métricas admitidas en Cognitive Services. Para ello, vaya a la sección Alertas y métricas de Azure Monitor.

Configure los valores de diagnóstico para el contenedor de Cognitive Services y envíe registros a un área de trabajo de Log Analytics. En el área de trabajo Log Analytics, configure las alertas que se activarán cuando se cumpla un conjunto predefinido de condiciones. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Creación, visualización y administración de alertas de registro mediante Azure Monitor](/azure/azure-monitor/platform/alerts-log)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para más información, consulte [Azure Security Benchmark: identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: Azure Active Directory (Azure AD) tiene roles integrados que se deben asignar explícitamente y son consultables. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc para detectar cuentas que son miembros de grupos administrativos.

- [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: el acceso del plano de control a Cognitive Services se controla mediante Azure Active Directory (Azure AD). Azure AD no tiene el concepto de contraseñas predeterminadas.

El acceso del plano de datos a Cognitive Services se controla mediante claves de acceso. Estas claves las utilizan los clientes al conectarse a la memoria caché y se pueden volver a generar en cualquier momento.

No se recomienda crear contraseñas predeterminadas en la aplicación. En su lugar, puede almacenar sus contraseñas en Azure Key Vault y, a continuación, usar Azure AD para recuperarlas.

- [Regeneración de claves de acceso de Azure Cache for Redis](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-configure#settings)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidades y acceso de Security Center para supervisar el número de cuentas administrativas.

Además, para ayudarle a realizar un seguimiento de las cuentas administrativas dedicadas, puede seguir las recomendaciones de Security Center o las directivas integradas de Azure, por ejemplo:

- Debe haber más de un propietario asignado a su suscripción
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción
- Las cuentas externas con permisos de propietario deben quitarse de la suscripción

- [Uso de Azure Security Center para supervisar la identidad y el acceso (versión preliminar)](../security-center/security-center-identity-access.md)

- [Uso de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4: Uso del inicio de sesión único (SSO) de Azure Active Directory

**Guía**: Azure Cognitive Services usa claves de acceso para autenticar a los usuarios y no admite el inicio de sesión único (SSO) en el nivel del plano de datos. El acceso al plano de control de Cognitive Services está disponible mediante la API REST y admite SSO. Para autenticarse, establezca el encabezado de autorización de las solicitudes en JSON (notación de objetos JavaScript) Web Token, que se obtiene de Azure Active Directory.

- [Descripción de la API REST de Azure Cognitive Services](/rest/api/cognitiveservices/)

- [Descripción del SSO con Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: habilite la autenticación multifactor de Azure Active Directory (Azure AD) y siga las recomendaciones de administración de identidades y acceso de Security Center.

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6: Uso de estaciones de trabajo seguras y administradas por Azure para realizar tareas administrativas

**Guía**: utilice estaciones de trabajo de acceso con privilegios (PAW) que tengan configurada la autenticación multifactor para iniciar sesión en los recursos de Azure y configurarlos. 

- [Más información sobre las estaciones de trabajo con privilegios de acceso](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: use Privileged Identity Management (PIM) de Azure Active Directory (Azure AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno.

Además, use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

- [Cómo implementar Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Información sobre las detecciones de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: configure ubicaciones con nombre en el acceso condicional de Azure Active Directory para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP, o países y regiones concretos.

- [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización. Azure AD protege los datos mediante el cifrado seguro para datos en reposo y en tránsito, y también sales, hashes y almacena de forma segura las credenciales de usuario. Si el caso de uso admite l autenticación de AD, use Azure AD para autenticar las solicitudes a la API de Cognitive Services. 

Actualmente, solo Computer Vision API, Face API, Text Analytics API, Lector inmersivo, Form Recognizer, Anomaly Detector y todos los servicios de Bing, excepto Bing Custom Search, admiten la autenticación mediante Azure AD.

- [Autenticación de solicitudes en Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/authentication#authenticate-with-azure-active-directory)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Instrucciones**: Azure Active Directory (Azure AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, el cliente puede usar las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo los usuarios activos tengan acceso continuado.

Los clientes que mantienen el inventario de cuentas de usuario de API Management, concilian el acceso según sea necesario. En Administración de API, los desarrolladores son los usuarios de las API que se exponen mediante Administración de API. De forma predeterminada, las cuentas de desarrollador recién creadas se encuentran en estado activo y se asocian al grupo de desarrolladores. Las cuentas de desarrollador que se encuentran en estado activo se pueden usar para acceder a todas las API para las que tienen suscripciones.

- [Administración de cuentas de usuario en Azure API Management](../api-management/api-management-howto-create-or-invite-developers.md)

- [Obtención de una lista de usuarios de API Management](/powershell/module/az.apimanagement/get-azapimanagementuser)

- [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: tiene acceso a los orígenes del registro de eventos de las actividades de inicio de sesión, auditoría y riesgo de Azure Active Directory (Azure AD), que permiten la integración en Azure Sentinel o en una herramienta SIEM de terceros.

Para simplificar este proceso, cree una configuración de diagnóstico para las cuentas de usuario de Azure AD y envíe los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas de registro deseadas en Log Analytics.

- [Integración de los registros de actividad de Azure en Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12: Alerta de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: para desviaciones del comportamiento de inicio de sesión de cuenta, use las características de Azure Active Directory (Azure AD) Identity Protection y de detección de riesgos para configurar respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. También puede hacer que Azure Sentinel ingiera los datos para investigarlos más.

- [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: no disponible en Cognitive Services. La Caja de seguridad del cliente todavía no se admite en Cognitive Services.

- [Lista de servicios admitidos por la Caja de seguridad del cliente](https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-protection"></a>Protección de datos

*Para más información, consulte [Azure Security Benchmark: protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Guía**: implemente suscripciones o grupos de administración independientes para los entornos de desarrollo, pruebas y producción. Los recursos deben separarse por redes virtuales o subredes, etiquetarse adecuadamente y protegerse mediante un grupo de seguridad de red o Azure Firewall. Los recursos que almacenan o procesan datos confidenciales deben estar suficientemente aislados. Para las máquinas virtuales que almacenan o procesan datos confidenciales, implemente la directiva y los procedimientos necesarios para desactivarlos cuando no estén en uso.

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

- [Creación de una red virtual](../virtual-network/quick-create-portal.md)

- [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

- [Implementación de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)

- [Configuración de alertas o alertas y denegación con Azure Firewall](../firewall/threat-intel.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: no disponible todavía. las características de identificación, clasificación y prevención de pérdida de datos no están disponibles aún en Cognitive Services.

Microsoft administra la infraestructura subyacente para Cognitive Services y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: todos los puntos de conexión de Cognitive Services se exponen a través de HTTP y aplican TLS 1.2. Con un protocolo de seguridad aplicado, los consumidores que intentan llamar a un punto de conexión de Cognitive Services deben cumplir estas directrices:

- El sistema operativo (SO) del cliente debe admitir TLS 1.2.
- El lenguaje (y la plataforma) que se usen para hacer la llamada HTTP tienen que especificar TLS 1.2 como parte de la solicitud. En función del lenguaje y la plataforma, la especificación de TLS se realiza implícita o explícitamente.

- [Descripción de la seguridad de la capa de transporte para Azure Cognitive Services](cognitive-services-security.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: las características de identificación, clasificación y prevención de pérdida de datos no están disponibles aún en Cognitive Services. Etiquete las instancias que contienen información confidencial como tal e implemente una solución de terceros, si es necesario, para fines de cumplimiento.

Microsoft administra la plataforma subyacente, trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos 

**Guía**: utilice el control de acceso basado en rol de Azure (Azure RBAC) para controlar el acceso al plano de control de Cognitive Services (por ejemplo, Azure Portal). 

- [Configuración de Azure RBAC](../role-based-access-control/role-assignments-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: el cifrado en reposo para Cognitive Services depende del servicio específico que se use. En la mayoría de los casos, los datos se cifran y descifran con el cifrado AES de 256 bits compatible con FIPS 140-2. El cifrado y el descifrado son transparentes, lo que significa que Microsoft administra el cifrado y el acceso para los clientes. Los datos de los clientes están protegidos de forma predeterminada y no es necesario modificar el código ni las aplicaciones para utilizar el cifrado.

También puede usar Azure Key Vault para almacenar las claves administradas por el cliente. Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generarlas.

- [Lista de servicios que cifran información en reposo](/azure/cognitive-services/encryption/cognitive-services-encryption-keys-portal)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: use Azure Monitor con el registro de actividad de Azure a fin de crear alertas para cuando se produzcan cambios en las instancias de producción de Cognitive Services y otros recursos críticos o relacionados.

- [Creación de alertas para los eventos del registro de actividad de Azure](/azure/azure-monitor/platform/alerts-activity-log)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para más información, consulte [Azure Security Benchmark: inventario y administración de recursos](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: utilice Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, puertos, protocolos, etc.) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y enumere todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

- [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription)

- [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Instrucciones**: Aplique etiquetas a los recursos de Azure que proporcionan metadatos para organizarlos de forma lógica en una taxonomía.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de las instancias de Azure Cache for Redis y recursos relacionados. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

Además, use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Además, use Azure Resource Graph para consultar o detectar recursos dentro de las suscripciones.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#general)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Guía**: Configure el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management.

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Azure Security Benchmark: configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: defina e implemente configuraciones de seguridad estándar para el contenedor de Cognitive Services con Azure Policy. Use alias de Azure Policy en el espacio de nombres "Microsoft.CognitiveServices" para crear directivas personalizadas a fin de auditar o aplicar la configuración de las instancias de Azure Cache for Redis.

- [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: use las directivas [deny] y [deploy if not exist] de Azure Policy para aplicar una configuración segura en los recursos de Azure.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: si usa definiciones de Azure Policy personalizadas o plantillas de Azure Resource Manager para los contenedores y recursos relacionados de Azure Cognitive Services, use Azure Repos para almacenar y administrar el código de forma segura.

- [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [Documentación de Azure Repos](/azure/devops/repos/)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: Use alias de Azure Policy en el espacio de nombres "Microsoft.Cache" para crear directivas personalizadas a fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Instrucciones**: use alias de Azure Policy en el espacio de nombres "Microsoft.CognitiveServices" para crear definiciones personalizadas de Azure Policy que permitan auditar y aplicar las configuraciones, así como enviar alertas sobre ellas. Use los efectos [audit], [deny] y [deploy if not exist] para aplicar automáticamente las configuraciones de las instancias y los recursos relacionados de Azure Cache for Redis.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: en el caso de las máquinas virtuales de Azure o las aplicaciones web que se ejecutan en Azure App Service y se usan para acceder a la API de Cognitive Services, utilice Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de claves de Cognitive Services. Asegúrese de que la eliminación temporal de Key Vault esté habilitada.

- [Integración con identidades administradas de Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

- [Creación de un almacén de claves](/azure/key-vault/quick-create-portal)

- [Autenticación en Azure Key Vault](../key-vault/general/authentication.md)

- [Asignación de una directiva de acceso de Key Vault](../key-vault/general/assign-access-policy-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: en el caso de las máquinas virtuales de Azure o las aplicaciones web que se ejecutan en Azure App Service y se usan para acceder a la API de Cognitive Services, utilice Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de claves de Cognitive Services. Asegúrese de que la eliminación temporal de Key Vault esté habilitada.

Use identidades administradas para proporcionar servicios de Azure con una identidad administrada automáticamente en Azure Active Directory (Azure AD). Identidades administradas le permite autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluido Azure Key Vault, sin necesidad de credenciales en el código.

- [Configuración de las identidades administradas](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

- [Integración con identidades administradas de Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

- [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Azure Security Benchmark: defensa contra malware](../security/benchmarks/security-control-malware-defense.md).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Cache for Redis), pero no se ejecuta en el contenido del cliente.

Examine previamente el contenido que se carga en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, Azure Database for PostgreSQL, etc. Microsoft no tiene acceso a los datos de estas instancias.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Azure Security Benchmark: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Los datos de la cuenta de Microsoft Azure Storage siempre se replican automáticamente para garantizar la durabilidad y la alta disponibilidad. Azure Storage copia los datos para protegerlos de eventos previstos e imprevistos, como errores transitorios del hardware, interrupciones del suministro eléctrico o de la red y desastres naturales masivos. Puede optar por replicar los datos en el mismo centro de datos, en centros de datos zonales que estén en la misma región o en regiones geográficamente separadas. 

También puede usar la característica de administración del ciclo de vida para realizar copias de seguridad de los datos en el nivel de archivo. Además, habilite la eliminación temporal para las copias de seguridad almacenadas en la cuenta de almacenamiento.

- [Descripción de la redundancia de Azure Storage y los acuerdos de nivel de servicio](../storage/common/storage-redundancy.md)

- [Administración del ciclo de vida de Azure Blob Storage](../storage/blobs/storage-lifecycle-management-concepts.md)

- [Eliminación temporal de blobs de Azure Storage](../storage/blobs/soft-delete-blob-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Instrucciones**: use Azure Resource Manager para implementar Cognitive Services y los recursos relacionados. Azure Resource Manager ofrece la posibilidad de exportar plantillas, lo que le permite volver a implementar la solución a lo largo de su ciclo de vida y tener la seguridad de que los recursos se implementan de forma coherente. Use Azure Automation para llamar a la API de exportación de plantillas de Azure Resource Manager de forma periódica. Realice una copia de seguridad de las claves precompartidas en Azure Key Vault.

- [Introducción sobre Azure Resource Manager](../azure-resource-manager/management/overview.md)

- [Creación de un recurso de Cognitive Services con una plantilla de Azure Resource Manager](https://docs.microsoft.com/azure/cognitive-services/resource-manager-template?tabs=portal)

- [Exportación de uno y varios recursos a una plantilla en Azure Portal](../azure-resource-manager/templates/export-template-portal.md)

- [Grupos de recursos: Exportar plantilla](/rest/api/resources/resourcegroups/exporttemplate)

- [Introducción a Azure Automation](../automation/automation-intro.md)

- [Creación de una copia de seguridad de las claves del almacén de claves en Azure](/powershell/module/az.keyvault/backup-azkeyvaultkey)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: en caso necesario, asegúrese de poder realizar periódicamente la implementación de plantillas de Azure Resource Manager de forma habitual en una suscripción aislada. Pruebe la restauración de copias de seguridad de las claves precompartidas.

- [Implementación de recursos con Azure Portal y plantillas de Resource Manager](../azure-resource-manager/templates/deploy-portal.md)

- [Restauración de las claves del almacén de claves en Azure](/powershell/module/az.keyvault/restore-azkeyvaultkey)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas del cliente

**Guía**: use Azure DevOps para almacenar y administrar de forma segura sus plantillas de Azure Resource Manager. Para proteger los recursos que administra en Azure DevOps, puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integran con Azure DevOps, o en Active Directory si se integran con Team Foundation Server.

Use el control de acceso basado en roles de Azure para proteger las claves administradas por el cliente. Habilite la eliminación temporal y la protección de purga en Key Vault para proteger las claves contra la eliminación accidental o malintencionada. 

- [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [Acerca de los permisos y los grupos en Azure DevOps](/azure/devops/organizations/security/about-permissions)

- [Eliminación temporal de blobs de Azure Storage](../storage/blobs/soft-delete-blob-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Azure Security Benchmark: respuesta ante incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

- [Configuración de las automatizaciones de flujos de trabajo en Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process)

- [También puede utilizar la guía de control de incidentes de seguridad de equipos de NIST como referencia para la creación de su propio plan de respuesta a incidentes](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta. 

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

- [Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente.  Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

- [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Guía**: exporte sus alertas y recomendaciones de Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Security Center para transmitir las alertas a Azure Sentinel.

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

**Guía**: Siga las reglas de compromiso de la prueba de penetración de Microsoft Cloud para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft. 

- [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [Información general sobre Azure Security Benchmark V2](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
