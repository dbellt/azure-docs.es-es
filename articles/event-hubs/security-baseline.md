---
title: Línea base de seguridad de Azure para Event Hubs
description: La línea base de seguridad de Event Hubs proporciona instrucciones de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: event-hubs
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 33417a9bda9ad4ce36dd6e14f74a53911f3c3473
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107587160"
---
# <a name="azure-security-baseline-for-event-hubs"></a>Línea base de seguridad de Azure para Event Hubs

Esta línea base de seguridad aplica las guías de [Azure Security Benchmark, versión 1.0](../security/benchmarks/overview-v1.md) a Event Hubs. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure.
El contenido se agrupa por medio de los **controles de seguridad** definidos por Azure Security Benchmark y las directrices relacionadas aplicables a Event Hubs. Se han excluido los **controles** no aplicables a Event Hubs.

 
Para ver cómo Event Hubs se asigna por completo a Azure Security Benchmark, consulte el [archivo de asignación de base de referencia de seguridad de Event Hubs completo](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Seguridad de redes

*Para más información, consulte [Azure Security Benchmark: seguridad de red](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: La integración de Azure Event Hubs en puntos de conexión de servicio de red virtual permite el acceso seguro a las funcionalidades de mensajería desde cargas de trabajo, tales como máquinas virtuales que están enlazadas a redes virtuales, con una ruta de acceso del tráfico de red que está protegida en ambos extremos.

Una vez que se enlaza con al menos un punto de conexión de servicio de subred de red virtual, el espacio de nombres respectivo de Event Hubs ya solo aceptará el tráfico procedente de redes virtuales autorizadas. Desde la perspectiva de la red virtual, el enlace del espacio de nombres de Event Hubs a un punto de conexión de servicio configura un túnel de redes aislado desde la subred de la red virtual al servicio de mensajería. 

También puede crear un punto de conexión privado, que es una interfaz de red que lo conecta de forma privada y segura al servicio Event Hubs mediante el servicio Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual para incorporar el servicio de manera eficaz a su red virtual. Todo el tráfico dirigido al servicio se puede enrutar mediante el punto de conexión privado, por lo que no se necesita ninguna puerta de enlace, dispositivos NAT, conexiones de ExpressRoute o VPN ni direcciones IP públicas. 

También puede proteger el espacio de nombres de Azure Event Hubs mediante firewalls. Event Hubs admite controles de acceso basado en IP para la compatibilidad con el firewall de entrada. Puede establecer reglas de firewall mediante Azure Portal, plantillas de Azure Resource Manager, o a través de la CLI de Azure o Azure PowerShell.

- [Permitir el acceso al espacio de nombres de Event Hubs desde redes virtuales específicas](event-hubs-service-endpoints.md)

- [Permiso para acceder a los espacios de nombres de Azure Event Hubs a través de puntos de conexión privados](private-link-service.md)

- [Permitir el acceso a los espacios de nombres de Azure Event Hubs desde intervalos o direcciones IP específicas](event-hubs-ip-filtering.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.EventHub**:

[!INCLUDE [Resource Policy for Microsoft.EventHub 1.1](../../includes/policy/standards/asb/rp-controls/microsoft.eventhub-1-1.md)]

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes e interfaces de red

**Instrucciones**: Use Azure Security Center y siga las recomendaciones de protección de redes para ayudar a proteger sus recursos de Event Hubs en Azure. Si usa máquinas virtuales de Azure para obtener acceso a los centros de eventos, habilite los registros de flujo de grupos de seguridad de red y envíe los registros a una cuenta de almacenamiento para la auditoría de tráfico.

- [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Descripción de la seguridad de red proporcionada por Azure Security Center](../security-center/security-center-network-recommendations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Denegación de las comunicaciones con direcciones IP malintencionadas conocidas

**Instrucciones**: Habilite el estándar de DDoS Protection en las redes virtuales asociadas a los centros de eventos para protegerse de los ataques por denegación de servicio distribuidos (DDoS). Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

- [Configuración de la protección contra DDoS](../ddos-protection/manage-ddos-protection.md)

- [Para obtener más información sobre la inteligencia sobre amenazas integrada de Azure Security Center](../security-center/azure-defender.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Guía**: Si usa máquinas virtuales de Azure para obtener acceso a los centros de eventos, habilite los registros de flujo de grupos de seguridad de red y envíe los registros a una cuenta de almacenamiento para la auditoría de tráfico. También puede enviar registros de flujo del grupo de seguridad de red a un área de trabajo de Log Analytics y usar Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

Si es necesario para investigar actividades anómalas, habilite la captura de paquetes de Network Watcher.

- [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

- [Habilitación de Network Watcher](../network-watcher/network-watcher-create.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implementación de sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Instrucciones**: Si usa máquinas virtuales de Azure para obtener acceso a los centros de eventos, seleccione una oferta de Azure Marketplace que admita la funcionalidad de IDS/IPS con funcionalidades de inspección de carga. Si la detección o prevención de intrusiones basadas en la inspección de la carga no es necesaria en su organización, puede usar la característica de firewall integrada de Azure Event Hubs. Puede restringir el acceso al espacio de nombres de Event Hubs para un intervalo limitado de direcciones IP o una dirección IP específica mediante reglas de firewall.

- [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

- [Permitir el acceso a los espacios de nombres de Azure Event Hubs desde intervalos o direcciones IP específicas](event-hubs-ip-filtering.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: Defina e implemente configuraciones de seguridad estándar para los recursos de red asociados a los espacios de nombres de Azure Event Hubs con Azure Policy. Use alias de Azure Policy en los espacios de nombres **Microsoft.EventHub** y **Microsoft.Network** para crear definiciones de directivas personalizadas con el fin de auditar o aplicar la configuración de red de los espacios de nombres de Event Hubs. También puede usar definiciones de directivas integradas relacionadas con Azure Event Hubs, como:

- El centro de eventos debe usar un punto de conexión del servicio de red virtual.

Información adicional disponible en los vínculos indicados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Definiciones de directiva integradas de Azure Policy](../governance/policy/samples/built-in-policies.md#event-hub)

- [Ejemplos de Azure Policy para redes](../governance/policy/samples/built-in-policies.md#network)

- [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: Use etiquetas para redes virtuales y otros recursos relacionados con la seguridad de red y el flujo de tráfico que están asociados a los centros de eventos.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: Use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en los recursos de red relacionados con Azure Event Hubs. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

- [Visualización y recuperación de eventos del registro de actividad de Azure](../azure-monitor/essentials/activity-log.md#view-the-activity-log)

- [Creación de alertas en Azure Monitor](../azure-monitor/alerts/alerts-activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Azure Security Benchmark: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Instrucciones**: Dentro de Azure Monitor, configure los registros relacionados con Event Hubs en el registro de actividad y la configuración de diagnóstico de Event Hubs para enviar los registros a un área de trabajo de Log Analytics que se va a consultar o a una cuenta de almacenamiento para el almacenamiento de archivo a largo plazo.

- [Configuración de registros de diagnóstico de Azure Event Hubs](event-hubs-diagnostic-logs.md)

- [Descripción del registro de actividad de Azure](../azure-monitor/essentials/platform-logs-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Habilite la configuración de diagnóstico para el espacio de nombres de Azure Event Hubs. Existen tres categorías de configuración de diagnóstico para Azure Event Hubs: Registros de archivo, registros operativos y registros de escalabilidad automática. Habilite los registros operativos para capturar información sobre lo que sucede durante las operaciones de Event Hubs, en concreto, el tipo de operación, como la creación del centro de eventos, los recursos usados y el estado de la operación.

Además, puede habilitar la configuración de diagnóstico del registro de actividad de Azure y enviarla a una cuenta de Azure Storage, un centro de eventos o un área de trabajo de Log Analytics. Los registros de actividades proporcionan información sobre las operaciones llevadas a cabo en Azure Event Hubs y otros recursos. Con los registros de actividades, se puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en los espacios de nombres de Azure Event Hubs.

- [Configuración de registros de diagnóstico de Azure Event Hubs](event-hubs-diagnostic-logs.md)

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/essentials/activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.EventHub**:

[!INCLUDE [Resource Policy for Microsoft.EventHub 2.3](../../includes/policy/standards/asb/rp-controls/microsoft.eventhub-2-3.md)]

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización, con el fin de capturar y revisar los incidentes relacionados con los centros de eventos.

- [Procedimiento para configurar parámetros de retención de registros en áreas de trabajo de Log Analytics](../azure-monitor/logs/manage-cost-storage.md#change-the-data-retention-period)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Guía**: Analice y supervise los registros en busca de comportamientos anómalos y revise periódicamente los resultados relacionados con los centros de eventos. Use Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos del registro. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una solución de administración de eventos e información de un sistema de terceros.

- [Más información sobre el área de trabajo de Log Analytics](../azure-monitor/logs/log-analytics-tutorial.md)

- [Procedimiento para realizar consultas personalizadas en Azure Monitor](../azure-monitor/logs/get-started-queries.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Dentro de Azure Monitor, configure los registros relacionados con Azure Event Hubs en el registro de actividad y la configuración de diagnóstico de Event Hubs para enviar los registros a un área de trabajo de Log Analytics que se va a consultar o a una cuenta de almacenamiento para el almacenamiento de archivo a largo plazo. Use el área de trabajo de Log Analytics para crear alertas para actividades anómalas que se encuentren en registros y eventos de seguridad.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel. 

- [Introducción a los registros de plataforma Azure](../azure-monitor/essentials/platform-logs-overview.md)

- [Configuración de registros de diagnóstico de Azure Event Hubs](event-hubs-diagnostic-logs.md)

- [Alertas sobre datos de registro del área de trabajo de Log Analytics](../azure-monitor/alerts/tutorial-response.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

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

**Guía**: El acceso del plano de control a Event Hubs se controla a través de Azure Active Directory (Azure AD). Azure AD no tiene el concepto de contraseñas predeterminadas.

El acceso del plano de datos a Event Hubs se controla a través de Azure AD con Identidades administradas o Registros de aplicaciones, así como con firmas de acceso compartido. Los clientes que se conectan a los centros de eventos usan las firmas de acceso compartido, que se pueden volver a generar en cualquier momento.

- [Autenticación del acceso a recursos de Event Hubs mediante firmas de acceso compartido (SAS)](authenticate-shared-access-signature.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

Además, para ayudarle a realizar un seguimiento de las cuentas administrativas dedicadas, puede seguir las recomendaciones de Azure Security Center o las directivas integradas de Azure Policy, por ejemplo:

- Debe haber más de un propietario asignado a su suscripción

- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción

- Las cuentas externas con permisos de propietario deben quitarse de la suscripción

Información adicional disponible en los vínculos indicados.

- [Uso de Azure Security Center para supervisar la identidad y el acceso (versión preliminar)](../security-center/security-center-identity-access.md)

- [Uso de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4: Uso del inicio de sesión único (SSO) de Azure Active Directory

**Guía**: Microsoft Azure proporciona una administración integrada del control de acceso para recursos y aplicaciones basados en Azure Active Directory (Azure AD). Una ventaja clave de usar Azure AD con Azure Event Hubs es que ya no es necesario almacenar las credenciales en el código. En su lugar, puede solicitar un token de acceso de OAuth 2.0 desde la Plataforma de identidad de Microsoft. El nombre del recurso para solicitar un token es https:\//eventhubs.azure.net/. Azure AD autentica la entidad de seguridad (un usuario, grupo o entidad de servicio) ejecutando la aplicación. Si la autenticación se realiza correctamente, Azure AD devuelve un token de acceso a la aplicación y la aplicación puede entonces usar el token de acceso para autorizar una solicitud de los recursos de Azure Event Hubs.

- [Autenticación de una aplicación con Azure Active Directory para acceder a recursos de Event Hubs](authenticate-application.md)

- [Descripción de SSO con Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Habilite la autenticación multifactor de Azure Active Directory (Azure AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center para ayudar a proteger los recursos habilitados para Event Hubs.

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: Utilice estaciones de trabajo con acceso con privilegios (PAW) con la autenticación multifactor configurada para iniciar sesión en recursos habilitados para Event Hubs y configurarlos.

- [Más información sobre las estaciones de trabajo con privilegios de acceso](/security/compass/privileged-access-devices)

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: Utilice Privileged Identity Management de Azure Active Directory (Azure AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno. Use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo. Para obtener más registros, envíe alertas de detección de Azure Security Center riesgo en Azure Monitor y configure alertas o notificaciones personalizadas con grupos de acciones.

- [Información sobre las detecciones de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Configuración de grupos de acciones para alertas y notificaciones personalizadas](../azure-monitor/alerts/action-groups.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: Use ubicaciones con nombre de acceso condicional para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

- [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: Use Azure Active Directory (Azure AD) como sistema central de autenticación y autorización para recursos de Azure, como Event Hubs. Esto facilita el control de acceso basado en roles de Azure (Azure RBAC) para administrar recursos confidenciales.

- [Inicio rápido: Creación de un inquilino en Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

- [Autorización del acceso a recursos de Event Hubs mediante Azure AD](authorize-access-azure-active-directory.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure Active Directory (Azure AD) proporciona registros para ayudarle a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

Además, rote periódicamente las firmas de acceso compartido de Event Hubs.

- [Descripción de los informes de Azure AD](../active-directory/reports-monitoring/index.yml)

- [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

- [Autenticación del acceso a recursos de Event Hubs mediante firmas de acceso compartido (SAS)](authenticate-shared-access-signature.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: Tiene acceso a los orígenes de registro de eventos de actividad de inicio de sesión de Azure Active Directory (Azure AD), auditoría y riesgo, que permiten la integración con cualquier solución de terceros de administración de eventos e información del sistema o una herramienta de supervisión.

Para simplificar este proceso, cree una configuración de diagnóstico para las cuentas de usuario de Azure AD y envíe los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas de registro deseadas en Log Analytics.

- [Integración de los registros de actividad de Azure en Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Autorización del acceso a recursos de Event Hubs mediante Azure AD](authorize-access-azure-active-directory.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12: Alerta de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Use las características Identity Protection y de detección de riesgos de Azure Active Directory (Azure AD) para configurar respuestas automatizadas a acciones sospechosas detectadas relacionadas con los recursos habilitados para Event Hubs. Debe habilitar las respuestas automatizadas a través de Azure Sentinel para implementar las respuestas de seguridad de su organización.

- [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Instrucciones**: Actualmente no disponible; la Caja de seguridad del cliente todavía no se admite en Event Hubs.

- [Lista de servicios admitidos por la Caja de seguridad del cliente](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-protection"></a>Protección de datos

*Para más información, consulte [Azure Security Benchmark: protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: Use etiquetas en los recursos relacionados con las instancias de Event Hubs para ayudar a realizar un seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Guía**: Implemente suscripciones independientes, opcionalmente con grupos de administración para desarrollo, pruebas y producción. Los espacios de nombres de Event Hubs deben estar separados por una red virtual con puntos de conexión de servicio habilitados y etiquetados adecuadamente.

También puede proteger el espacio de nombres de Azure Event Hubs mediante firewalls. Azure Event Hubs admite controles de acceso basado en IP para la compatibilidad con el firewall de entrada. Puede establecer reglas de firewall mediante Azure Portal, plantillas de Azure Resource Manager, o a través de la CLI de Azure o Azure PowerShell.

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Permitir el acceso a los espacios de nombres de Azure Event Hubs desde intervalos o direcciones IP específicas](event-hubs-ip-filtering.md)

- [Procedimiento para crear y usar etiquetas](../azure-resource-manager/management/tag-resources.md)

- [Creación de una red virtual](../virtual-network/quick-create-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Al usar máquinas virtuales para tener acceso a los centros de eventos, aproveche las redes virtuales, los puntos de conexión de servicio, el firewall de Event Hubs, los grupos de seguridad de red y las etiquetas de servicio para mitigar la posibilidad de que se produzca una filtración de datos.

Microsoft administra la infraestructura subyacente para Azure Event Hubs y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

- [Permitir el acceso a los espacios de nombres de Azure Event Hubs desde intervalos o direcciones IP específicas](event-hubs-ip-filtering.md)

- [Permitir el acceso al espacio de nombres de Event Hubs desde redes virtuales específicas](event-hubs-service-endpoints.md)

- [Permiso para acceder a los espacios de nombres de Azure Event Hubs a través de puntos de conexión privados](private-link-service.md)

- [Descripción de los grupos de seguridad de red y las etiquetas de servicio](../virtual-network/network-security-groups-overview.md)

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Instrucciones**: Las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Event Hubs. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6: Uso del control de acceso basado en rol para controlar el acceso a los recursos

**Guía**: Azure Event Hubs admite la autorización de solicitudes a recursos de Event Hubs mediante Azure Active Directory (Azure AD). Con Azure AD, puede usar el control de acceso basado en roles (RBAC) de Azure para conceder permisos a una entidad de seguridad, que puede ser un usuario o una entidad de servicio de aplicación.

- [Autorización del acceso a recursos de Event Hubs mediante Azure Active Directory](authorize-access-azure-active-directory.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Azure Event Hubs admite la opción de cifrado de datos en reposo con claves administradas por Microsoft o claves administradas por el cliente. Esta característica permite crear, rotar, deshabilitar y revocar el acceso a las claves administradas por el cliente que se usan para cifrar datos en reposo de Azure Event Hubs.

- [Configuración de claves administradas por el cliente para cifrar datos en reposo de Azure Event Hubs mediante Azure Portal](configure-customer-managed-key.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en las instancias de producción de Azure Event Hubs y otros recursos críticos o relacionados.

- [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/alerts/alerts-activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para más información, consulte [Azure Security Benchmark: inventario y administración de recursos](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Use Azure Resource Graph para consultar y detectar todos los recursos (incluidos los espacios de nombres de Azure Event Hubs) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

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

**Guía**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los espacios de nombres de Azure Event Hubs y los recursos relacionados. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

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

- [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Información adicional disponible en los vínculos indicados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](../governance/policy/samples/built-in-policies.md#general)

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

**Guía**: Defina e implemente configuraciones de seguridad estándar para las implementaciones de Azure Event Hubs. Use alias de Azure Policy en el espacio de nombres **Microsoft.EventHub** para crear directivas personalizadas con el fin de auditar o aplicar las configuraciones. También puede usar definiciones de directivas integradas para Azure Event Hubs, como:

- Los registros de diagnóstico del centro de eventos deben estar habilitados

- El centro de eventos debe usar un punto de conexión del servicio de red virtual

Información adicional disponible en los vínculos indicados.

- [Definiciones de directiva integradas de Azure Policy](../governance/policy/samples/built-in-policies.md#event-hub)

- [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Use las directivas de Azure Policy [deny] y [deployifnotexist] para aplicar la configuración segura en los recursos habilitados para Event Hubs. 

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

 
- [Para más información, consulte el artículo sobre los efectos de Azure Policy](../governance/policy/concepts/effects.md).

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: Use alias de Azure Policy en el espacio de nombres "Microsoft.EventHub" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: Use alias de Azure Policy en el espacio de nombres "Microsoft.EventHub" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Use las directivas de Azure Policy [audit], [deny] y [deployifnotexist] para aplicar automáticamente las configuraciones en las implementaciones de Azure Event Hubs y los recursos relacionados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: En el caso de las máquinas virtuales de Azure o las aplicaciones web que se ejecutan en Azure App Service y se usan para tener acceso a los centros de eventos, use Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de firmas de acceso compartido para sus implementaciones de Azure Event Hubs. Asegúrese de que Key Vault esté configurado con la eliminación temporal habilitada.

- [Autenticación de una identidad administrada con Azure Active Directory para acceder a recursos de Event Hubs](./authenticate-managed-identity.md?tabs=latest)

- [Configuración de claves administradas por el cliente para cifrar datos en reposo de Azure Event Hubs mediante Azure Portal](configure-customer-managed-key.md)

- [Integración con identidades administradas de Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

- [Creación de un almacén de claves](../key-vault/secrets/quick-create-portal.md)

- [Cómo proporcionar la autenticación de Key Vault con una identidad administrada](../key-vault/general/assign-access-policy-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: En el caso de las máquinas virtuales de Azure o las aplicaciones web que se ejecutan en Azure App Service y se usan para tener acceso a los centros de eventos, use Managed Service Identity junto con Azure Key Vault para simplificar y proteger Azure Event Hubs. Asegúrese de que Key Vault esté configurado con la eliminación temporal habilitada.

Use identidades administradas para proporcionar servicios de Azure con una identidad administrada automáticamente en Azure Active Directory (Azure AD). Identidades administradas le permite autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluido Azure Key Vault, sin necesidad de credenciales en el código.

- [Autenticación de una identidad administrada con Azure Active Directory para acceder a recursos de Event Hubs](./authenticate-managed-identity.md?tabs=latest)

- [Configuración de claves administradas por el cliente para cifrar datos en reposo de Azure Event Hubs mediante Azure Portal](configure-customer-managed-key.md)

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

**Guía**: Examine previamente el contenido que se carga en recursos de Azure que no son de proceso, como Azure Event Hubs, App Service, Data Lake Storage, Blob Storage, Azure Database for PostgreSQL, etc. Microsoft no tiene acceso a los datos de estas instancias.

Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Cache for Redis), pero no se ejecuta en el contenido del cliente.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Azure Security Benchmark: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Instrucciones**: Configure la recuperación ante desastres geográfica para Azure Event Hubs. Cuando hay regiones de Azure completas o centros de datos (si no se utilizan zonas de disponibilidad) que experimentan un tiempo de inactividad, es crucial que el procesamiento de datos siga funcionando en otra región o centro de datos. De esta forma, la recuperación ante desastres geográfica y la replicación geográfica son características importantes para cualquier empresa. Azure Event Hubs admite tanto la recuperación ante desastres con localización geográfica como la replicación geográfica, en el nivel de espacio de nombres. 

- [Azure Event Hubs: recuperación ante desastres geográfica](./event-hubs-geo-dr.md#availability-zones)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Azure Event Hubs proporciona cifrado de datos en reposo con Azure Storage Service Encryption (Azure SSE). Event Hubs se basa en Azure Storage para almacenar los datos y, de forma predeterminada, todos los datos que se almacenan con Azure Storage se cifran mediante claves administradas por Microsoft. Si usa Azure Key Vault para almacenar claves administradas por el cliente, asegúrese de realizar copias de seguridad automatizadas periódicas de las claves.

Asegúrese de realizar copias de seguridad automatizadas periódicas de sus secretos de Key Vault con el siguiente comando de PowerShell: Backup-AzKeyVaultSecret

- [Configuración de claves administradas por el cliente para cifrar datos en reposo de Azure Event Hubs mediante Azure Portal](configure-customer-managed-key.md)

- [Realización de copias de seguridad de secretos de Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: Pruebe la restauración de la copia de seguridad de las claves administradas por el cliente.

- [Restauración de las claves del almacén de claves en Azure](/powershell/module/az.keyvault/restore-azkeyvaultkey)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: Habilite la eliminación temporal en Key Vault para proteger las claves contra la eliminación accidental o malintencionada. Azure Event Hubs requiere que las claves administradas por el cliente tengan configurada Eliminación temporal y No purgar.

Configure la eliminación temporal para la cuenta de almacenamiento de Azure que se usa para capturar datos de Event Hubs. Tenga en cuenta que esta característica aún no se admite para Azure Data Lake Storage Gen 2.

- [Habilitación de la eliminación temporal en Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

- [Configuración de un almacén de claves con claves](configure-customer-managed-key.md)

- [Eliminación temporal de blobs de Azure Storage](/azure/storage/blobs/soft-delete-blob-overview)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Azure Security Benchmark: respuesta ante incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Guía**: Asegúrese de que haya planes de respuesta ante incidentes escritos que definan los roles del personal, así como las fases de administración y gestión de los incidentes.

- [Configuración de las automatizaciones de flujos de trabajo en Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a las alertas, que le ayudan a priorizar el orden en el que asiste a cada alerta, de modo que cuando un recurso está en peligro, puede obtenerlo inmediatamente. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Guía**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

- [Consulte la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente.  Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas. 

- [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: Exporte sus alertas y recomendaciones de Azure Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel.

- [Configuración de la exportación continua](../security-center/continuous-export.md)

- [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Instrucciones**: use la característica de automatización del flujo de trabajo de Azure Security Center para desencadenar automáticamente respuestas mediante "Logic Apps" en las alertas y recomendaciones de seguridad.

- [Configuración de la automatización de flujo de trabajo y Logic Apps](../security-center/workflow-automation.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Azure Security Benchmark: Pruebas de penetración y ejercicios del equipo rojo](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: Siga las reglas de compromiso de la prueba de penetración de Microsoft Cloud para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de pruebas de penetración de sitios activos y ataques simulados en la infraestructura en la nube, los servicios y las aplicaciones administrados por Microsoft. 

- [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [Información general sobre Azure Security Benchmark V2](../security/benchmarks/overview.md).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](../security/benchmarks/security-baselines-overview.md).
