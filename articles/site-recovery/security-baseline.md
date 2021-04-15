---
title: Base de referencia de seguridad de Azure para Site Recovery
description: La base de referencia de seguridad de Site Recovery proporciona instrucciones de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: site-recovery
ms.topic: conceptual
ms.date: 03/29/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 429bb1ffdf40ed9906082e00d4ffd1156a4e5e0b
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105967853"
---
# <a name="azure-security-baseline-for-site-recovery"></a>Base de referencia de seguridad de Azure para Site Recovery

Esta línea de base de seguridad aplica las instrucciones de [Azure Security Benchmark, versión 1.0](../security/benchmarks/overview-v1.md) a Site Recovery. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa según los **controles de seguridad** definidos en Azure Security Benchmark y las directrices relacionadas aplicables a Site Recovery. Se han excluido los **controles** que no son aplicables a Site Recovery o cuya responsabilidad es de Microsoft.

Para ver cómo Site Recovery se asigna por completo a Azure Security Benchmark, consulte el [archivo completo de asignación de línea de base de seguridad de Site Recovery](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Seguridad de redes

*Para más información, consulte [Azure Security Benchmark: seguridad de red](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: Microsoft Azure Site Recovery no admite la implementación en una instancia de Azure Virtual Network. Configure el servicio Site Recovery con un punto de conexión privado de Azure para aplicar el uso de comunicaciones seguras a través de la red.

- [Compatibilidad con vínculos privados de Azure Site Recovery](azure-to-azure-how-to-enable-replication-private-endpoints.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: el servicio Site Recovery admite etiquetas de servicio, lo que permite a los clientes abrir el tráfico únicamente para servicios y puertos concretos. Los clientes deben permitir la etiqueta de servicio "AzureSiteRecovery" en su firewall o grupo de seguridad de red para permitir el acceso saliente al servicio Site Recovery.

- [Conectividad saliente mediante etiquetas de servicio](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-about-networking#outbound-connectivity-using-service-tags)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: Use etiquetas de recurso para grupos de seguridad de red y otros recursos relacionados con la seguridad de red y el flujo de tráfico. En el caso de las reglas de grupo de seguridad de red individual, use el campo "Descripción" para documentar las reglas que permiten el tráfico con origen y destino en una red. 

Incorpore cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor", para que todos los recursos se creen con etiquetas y para notificarle de los recursos no etiquetados existentes. 

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas. 

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

- [Creación de una red virtual de Azure](../virtual-network/quick-create-portal.md) 

- [Filtrado del tráfico de red con reglas de grupos de seguridad de red](../virtual-network/tutorial-filter-network-traffic.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: supervise todos los cambios en las configuraciones de recursos de red relacionados con el servicio Site Recovery mediante los registros de actividad de Azure. Cree alertas en Azure Monitor para recibir una notificación cuando se produzcan cambios en los recursos de red críticos de Site Recovery.

- [Visualización y recuperación de eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/essentials/activity-log#view-the-activity-log)

- [Crear, ver y administrar las alertas del registro de actividad mediante Azure Monitor](../azure-monitor/alerts/alerts-activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Azure Security Benchmark: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: habilite la configuración de diagnóstico del registro de actividad de Azure para registros de auditoría y envíe los registros a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para archivarlo.

Use los datos del registro de actividad de Azure para determinar las respuestas a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en sus recursos de Azure.

Ingiera los registros de Site Recovery en Azure Monitor para agregar los datos de seguridad generados. Dentro de Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y use cuentas de almacenamiento para el almacenamiento de archivos o a largo plazo. Asimismo, puede habilitar e incorporar datos en Azure Sentinel o en una solución de administración de eventos e información de seguridad (SIEM) de terceros.

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/essentials/activity-log.md)

- [Supervisión de Site Recovery con registros de Azure Monitor](monitor-log-analytics.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: habilite la configuración de diagnóstico del registro de actividad de Azure para registros de auditoría y envíe los registros a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para archivarlo. 

Use los datos del registro de actividad de Azure para determinar las respuestas a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) llevadas a cabo en sus recursos de Azure.

Ingiera los registros de Site Recovery con Azure Monitor para agregar los datos de seguridad generados. Dentro de Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y use cuentas de almacenamiento para el almacenamiento de archivos o a largo plazo. Habilite e incorpore datos en Azure Sentinel o en una solución de Administración de eventos e información de seguridad (SIEM) de terceros.

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/essentials/activity-log.md)

- [Supervisión de Site Recovery con registros de Azure Monitor](monitor-log-analytics.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: establezca el período de retención de registro para las áreas de trabajo de Log Analytics asociadas a los almacenes de Azure Recovery Services mediante Azure Monitor según las normativas de cumplimiento de su organización. 

- [Establecimiento de parámetros de retención de registros](https://docs.microsoft.com/azure/azure-monitor/logs/manage-cost-storage#change-the-data-retention-period)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: Habilite la configuración de diagnóstico en el registro de actividad de Azure y envíe los registros a un área de trabajo de Log Analytics. 

Realice consultas en Log Analytics para buscar términos, identificar tendencias, analizar patrones y conclusiones sobre los datos del registro de actividad recopilados en los almacenes de Recovery Services.

- [Supervisión de Site Recovery](site-recovery-monitor-and-troubleshoot.md)

- [Habilitación de la configuración de diagnóstico para el registro de actividad de Azure](../azure-monitor/essentials/activity-log.md)

- [Recopilación y análisis de registros de actividad de Azure en un área de trabajo de Log Analytics en Azure Monitor](../azure-monitor/essentials/activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: supervise las máquinas replicadas por Azure Site Recovery mediante los registros de Azure Monitor y Log Analytics. Use Log Analytics en Azure Monitor para escribir y probar consultas de registro y para analizar interactivamente esos datos de registro. Azure Monitor recopila registros de actividad y de recursos, junto con otros datos de supervisión. 

Visualice y consulte los resultados del registro y configure alertas para realizar acciones basadas en los datos supervisados. Configure alertas en un área de trabajo de Log Analytics en Azure Sentinel, ya que proporciona una solución de respuesta automatizada de orquestación de seguridad (SOAR). Esto permite crear soluciones automatizadas, como cuadernos de estrategias, para corregir problemas de seguridad. Cree alertas de registro personalizadas en el área de trabajo Log Analytics mediante Azure Monitor. 

- [Supervisión de Site Recovery](site-recovery-monitor-and-troubleshoot.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Creación, visualización y administración de alertas de registro mediante Azure Monitor](../azure-monitor/alerts/alerts-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para más información, consulte [Azure Security Benchmark: identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: No se asignan roles de manera predeterminada. Deben asignarse explícitamente según las necesidades de la empresa. Las asignaciones de roles se pueden comprobar con PowerShell, la CLI o Azure Active Directory (Azure AD) para detectar las cuentas que son miembros de los grupos administrativos.

- [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas. Use la identidad de Security Center y las características de administración de acceso para supervisar el número de cuentas administrativas.

Además, para ayudarlo a realizar un seguimiento de las cuentas administrativas dedicadas, siga las recomendaciones de Security Center o las directivas integradas de Azure, por ejemplo: 

- Debe haber más de un propietario asignado a su suscripción 
- Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción 

- Las cuentas externas con permisos de propietario deben quitarse de la suscripción

Cree un proceso para realizar un seguimiento del control de acceso e identidad para las cuentas administrativas y revíselo periódicamente.

- [Procedimiento para usar Azure Security Center para supervisar la identidad y el acceso](../security-center/security-center-identity-access.md)

- [Uso de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Guía**: use un registro de aplicaciones de Azure con una entidad de servicio para recuperar un token que se pueda usar para interactuar con los almacenes de Recovery Services mediante llamadas API.

- [Llamada a las API REST de Azure](/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

- [Registro de la aplicación cliente (entidad de servicio) con Azure Active Directory (Azure AD)](/rest/api/azure/#register-your-client-application-with-azure-ad)

- [Información de API de Azure Recovery Services](/rest/api/recoveryservices)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: habilite Multi-Factor Authentication de Azure Active Directory y siga las recomendaciones de identidad y acceso de Security Center.

- [Plan de una implementación de Azure AD Multi-Factor Authentication](../active-directory/authentication/howto-mfa-getstarted.md)

- [Supervisión de la identidad y el acceso](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: use una estación de trabajo segura administrada por Azure (también conocida como estación de trabajo de acceso con privilegios, o PAW) con Multi-Factor Authentication de Azure Active Directory para tareas administrativas y para realizar acciones con privilegios en los recursos de Site Recovery. 

- [Uso de estaciones de trabajo con privilegios de acceso](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Plan de una implementación de Azure AD Multi-Factor Authentication](../active-directory/authentication/howto-mfa-getstarted.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: use la característica Privileged Identity Management (PIM) de Azure Active Directory (Azure AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno.

Vea las alertas e informes sobre el comportamiento de riesgo de los usuarios con la característica de detección de riesgo de Azure AD.

- [Cómo implementar Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Información sobre las detecciones de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="38-manage-azure-resources-only-from-approved-locations"></a>3.8: Administración de los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: use ubicaciones con nombre de acceso condicional para permitir el acceso a Azure Portal solo desde agrupaciones lógicas específicas de intervalos de direcciones IP, regiones o países.
- [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: use Azure Active Directory (Azure AD) como sistema central de autenticación y autorización para Site Recovery. Azure AD protege los datos mediante el cifrado seguro para datos en reposo y en tránsito; además, cifra con sal, hashes y almacena de forma segura las credenciales de usuario.

- [Procedimiento para crear y configurar una instancia de Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: use los registros de Azure Active Directory (Azure AD) para ayudarle a detectar cuentas obsoletas.

Administre de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles con las revisiones de acceso e identidad de Azure AD.

Cree un proceso que revise el acceso de usuario de manera periódica para asegurarse de que solo los usuarios con revisiones de acceso completadas cuentan con acceso continuo.

- [Descripción de los informes de Azure AD](/azure/active-directory/reports-monitoring/)

- [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: use Azure Active Directory (Azure AD) como sistema central de autenticación y autorización para los recursos de Site Recovery. Azure AD protege los datos mediante el cifrado seguro para datos en reposo y en tránsito, y también sales, hashes y almacena de forma segura las credenciales de usuario.

Tiene acceso a los orígenes del registro de eventos de las actividades de inicio de sesión, auditoría y riesgo de Azure AD, que permiten integrarlos con Azure Sentinel o cualquier herramienta de supervisión o SIEM de terceros disponible en Azure Marketplace.

Simplifique aún más este proceso al crear una configuración de diagnóstico para las cuentas de usuario de Azure AD y enviar los registros de auditoría e inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas que quiera en un área de trabajo de Log Analytics.

- [Integración de los registros de actividad de Azure en Azure Monitor](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización para los almacenes de Recovery Services.

Haga uso de las características de protección de identidad de Azure AD para detectar comportamientos de inicio de sesión de las cuentas y para configurar respuestas automatizadas a las acciones sospechosas detectadas, según se relacionen con las identidades de los usuarios. Además, ingiera datos en Azure Sentinel para investigarlos más.

- [Visualización del inicio de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

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

**Guía**: implemente suscripciones o grupos de administración independientes para el desarrollo, prueba y producción de almacenes de Recovery Services. Separe los recursos mediante una red virtual o subred, etiquételos adecuadamente y protéjalos con un grupo de seguridad de red o Azure Firewall. 

Apague las máquinas virtuales que almacenan o procesan datos confidenciales cuando no estén en uso. Implemente la directiva y los procedimientos para realizar este proceso de manera periódica. 

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Introducción a Site Recovery](site-recovery-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: use un vínculo privado o un punto de conexión privado, grupos de seguridad de red y etiquetas de servicio para mitigar todas las oportunidades de filtración de datos de las máquinas virtuales habilitadas para Site Recovery.

Microsoft administra la plataforma subyacente que usa Site Recovery, trata todo el contenido de los clientes como confidencial y protege a los clientes contra la pérdida y exposición de sus datos. Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos para que los datos de los clientes que se encuentran en Azure estén protegidos. 

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

- [Replicación de máquinas virtuales con puntos de conexión privados de Azure](azure-to-azure-how-to-enable-replication-private-endpoints.md)

- [Replicación de máquinas virtuales con etiquetas de servicio de Azure Site Recovery](azure-to-azure-about-networking.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: Site Recovery usa un canal https seguro cifrado con el estándar de cifrado avanzado (AES 256) desde servidores de carga de trabajo de Azure a los servicios Site Recovery hospedados detrás de un almacén de Recovery Services.

Las versiones de TLS que actualmente son compatibles con Site Recovery son TLS 1.0, TLS 1.1, TLS 1.2 en regiones que estaban publicadas al final de 2019. TLS 1.2 es la única versión de TLS admitida para las nuevas regiones.

- [Descripción del cifrado en tránsito para Azure Site Recovery](physical-azure-set-up-source.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Site Recovery. 

Implemente una solución de terceros, según sea necesario, para fines de cumplimiento.

Microsoft administra la plataforma subyacente que usa Site Recovery, trata todo el contenido de los clientes como confidencial y protege a los clientes contra la pérdida y exposición de sus datos. Ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos para que los datos de los clientes que se encuentran en Azure estén protegidos. 

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4.6: Uso de RBAC de Azure para administrar el acceso a los recursos

**Guía**: use el control de acceso basado en roles de Azure (Azure RBAC) para administrar el acceso a los recursos y datos relacionados con los recursos de Site Recovery. 

Separe los deberes laborales con Azure RBAC y conceda el acceso adecuado necesario para realizarlos. Use los roles de Site Recovery integrados para controlar las operaciones de administración de Site Recovery.

- [Configuración de Azure RBAC](../role-based-access-control/role-assignments-portal.md)

- [Uso del control de acceso basado en roles para administrar Azure Site Recovery](site-recovery-role-based-linked-access-control.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: habilite el cifrado doble con claves administradas por el cliente y la plataforma. Esta funcionalidad está disponible en Site Recovery. 

Site Recovery admite el cifrado en reposo de los datos. Para las cargas de trabajo de Azure IaaS, los datos se cifran en reposo con Storage Service Encryption (SSE). 

Solo el cliente tiene acceso a la clave de cifrado mientras usa un almacén de Recovery Services cifrado con una clave administrada por el cliente. Microsoft nunca conserva una copia, ni tiene acceso a la clave y no descifra los datos transferidos desde la ubicación principal a la de recuperación ante desastres en cualquier momento. 

- [Compatibilidad con claves administradas por el cliente para Azure Site Recovery](azure-to-azure-how-to-enable-replication-cmk-disks.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: use Azure Monitor con los registros de actividad de Azure para crear alertas para cuando se produzcan cambios en los recursos críticos. Estos recursos podrían incluir instancias de producción de almacenes de Recovery Services, recursos del servicio de Site Recovery y recursos relacionados.
- [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/alerts/alerts-activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para más información, consulte [Azure Security Benchmark: inventario y administración de recursos](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: use Azure Resource Graph para consultar o detectar todos los recursos, incluidos los almacenes de Recovery Services, en las suscripciones. Asegúrese de que tiene los permisos de lectura adecuados en el inquilino y enumere todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

- [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

- [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription)

- [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: aplique etiquetas a los almacenes de Recovery Services y otros recursos relacionados, utilizados por Site Recovery con metadatos, para organizarlos lógicamente según una taxonomía.
- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de Site Recovery (almacenes de Recovery Services) y otros recursos relacionados. 

Además, use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas: 

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: cree un inventario de los recursos de Azure aprobados y el software aprobado para los recursos de proceso según los requisitos organizativos del cliente.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas: 

- Tipos de recursos no permitidos 
- Tipos de recursos permitidos

Use Azure Resource Graph para consultar y detectar recursos dentro de sus suscripciones.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos 
- Tipos de recursos permitidos

Comprender cómo se crean y administran las directivas en Azure es importante para mantener el cumplimiento de los estándares corporativos y los contratos de nivel de servicio.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](/azure/governance/policy/samples)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure". Esto puede impedir la creación y los cambios en los recursos de un entorno de alta seguridad.

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Azure Security Benchmark: configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Instrucciones**: Defina e implemente configuraciones de seguridad estándar para el almacén de Recovery Services con Azure Policy. 

Use alias de Azure Policy en el espacio de nombres "Microsoft.RecoveryServices" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de los recursos de almacenes de Recovery Services del servicio Site Recovery.
- [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Use las directivas [deny] y [deploy if not exist] de Azure Policy para aplicar una configuración segura en los recursos de Azure.
- [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: elija Azure Repos para almacenar y administrar código de manera segura si usa definiciones personalizadas de Azure Policy para los almacenes de Recovery Services y los recursos relacionados.

- [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [Documentación de Azure Repos](/azure/devops/repos/)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: Use las definiciones de Azure Policy integradas junto con los alias de Azure Policy en el espacio de nombres "Microsoft.RecoveryServices" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. 

Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: Use las definiciones de Azure Policy integradas junto con los alias de Azure Policy en el espacio de nombres "Microsoft.RecoveryServices" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. 

Utilice los efectos de la directiva de Azure Policy [audit], [deny] y [deploy if not exist] para aplicar automáticamente las configuraciones en los recursos de Azure.
- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: el cliente debe administrar los secretos de Site Recovery integrados con Azure Key Vault, a la vez que permite la recuperación ante desastres para máquinas virtuales habilitadas para Azure Disk Encryption. 

- [Creación de un almacén de claves](../key-vault/general/quick-create-portal.md)

- [Autenticación en Key Vault](../key-vault/general/authentication.md)

- [Asignación de una directiva de acceso de Key Vault](../key-vault/general/assign-access-policy-portal.md)

- [Habilitación de la recuperación ante desastres para máquinas virtuales habilitadas para Azure Disk Encryption mediante Site Recovery](azure-to-azure-how-to-enable-replication-ade-vms.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: Site Recovery admite las identidades administradas por el sistema solo cuando un cliente puede habilitar las identidades administradas por el sistema en el almacén de Recovery Services. La misma metodología se aplica a los recursos usados en la oferta de recuperación ante desastres para definir el límite de acceso.

Use identidades administradas para proporcionar servicios de Azure con una identidad administrada automáticamente en Azure Active Directory (Azure AD).

Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, como Key Vault, sin necesidad de credenciales en el código.

- [Integración con identidades administradas de Azure](https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity?tabs=core2x)

- [Habilitación de la identidad administrada por el sistema en el almacén de Recovery Services](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-how-to-enable-replication-private-endpoints#enable-the-managed-identity-for-the-vault)

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

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que da soporte a los servicios de Azure (por ejemplo, Site Recovery), pero no se ejecuta en el contenido. Examine previamente los archivos que se cargan en recursos de Azure que no son de proceso, como App Service, Data Lake Storage y Blob Storage.

Use la detección de amenazas de Security Center para los servicios de datos para detectar malware cargado en cuentas de almacenamiento.

- [Descripción de Microsoft Antimalware para Azure Cloud Services y Virtual Machines](../security/fundamentals/antimalware.md)

- [Descripción de la detección de amenazas de Azure Security Center para servicios de datos](../security-center/azure-defender.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Azure Security Benchmark: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Site Recovery usa internamente una cuenta de Azure Storage para mantener el estado de la solución de recuperación ante desastres, según lo hayan configurado configuración los clientes en sus cargas de trabajo.

Todos los recursos de almacenamiento utilizados por los metadatos de los servicios de Site Recovery con la configuración de tipo: almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS). Las cuentas de almacenamiento de un tipo superior a GRS (como RAGRS, RAG-ZRS) replican los datos en una región secundaria (a cientos de kilómetros de la ubicación principal de los datos de origen) para seguir atendiendo a los clientes de la recuperación ante desastres durante las interrupciones.

Esto se encuentra fuera del ámbito del cliente y el equipo de Site Recovery se encarga internamente. El cliente puede crear una copia de seguridad de las claves de Key Vault en Azure.

- [Creación de una copia de seguridad de las claves del almacén de claves en Azure](/powershell/module/az.keyvault/backup-azkeyvaultkey)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.RecoveryServices**:

[!INCLUDE [Resource Policy for Microsoft.RecoveryServices 9.2](../../includes/policy/standards/asb/rp-controls/microsoft.recoveryservices-9-2.md)]

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: pruebe periódicamente las restauraciones de la copia de seguridad de las claves administradas por el cliente.

- [Restauración de las claves del almacén de claves en Azure](/powershell/module/az.keyvault/restore-azkeyvaultkey)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: los datos se cifran en reposo mediante Storage Service Encryption (SSE) con la infraestructura como servicio (IaaS) de Azure basada en máquinas virtuales. Habilite la eliminación temporal en Key Vault para proteger las claves contra la eliminación accidental o malintencionada.

- [Habilitación de la eliminación temporal en Key Vault](https://docs.microsoft.com/azure/storage/blobs/soft-delete-blob-overview?tabs=azure-portal)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Azure Security Benchmark: respuesta ante incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. 

Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración o administración de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

- [Configuración de las automatizaciones de flujos de trabajo en Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process)

- [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process)

- [El cliente también puede usar la guía de control de incidentes de seguridad de equipos de NIST como referencia para la creación de su propio plan de respuesta a incidentes](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Guía**: clasifique por orden de prioridad las alertas que se deben investigar primero en función de la gravedad que Security Center asigna a cada alerta. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Marque las suscripciones de manera clara (por ejemplo, producción o no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure.

- [Alertas de seguridad en el Centro de seguridad de Azure](../security-center/security-center-alerts-overview.md) 

- [Uso de etiquetas para organizar los recursos de Azure](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

- [Consulte la publicación de NIST: "Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI"](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf).

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente. 

Cree un proceso para revisar los incidentes, después de la aparición, para asegurarse de que los problemas se resolvieron.

- [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Guía**: exporte sus alertas y recomendaciones de Security Center mediante la característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. 

Use el conector de datos de Security Center para transmitir las alertas a Azure Sentinel, según sea necesario.
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
