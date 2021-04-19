---
title: Línea de base de seguridad de Azure para VPN Gateway
description: La línea de base de seguridad de VPN Gateway proporciona instrucciones de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 04/08/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 2b27b30f45d31e8bf6f334106af1516e7fc4cf56
ms.sourcegitcommit: c6a2d9a44a5a2c13abddab932d16c295a7207d6a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107284177"
---
# <a name="azure-security-baseline-for-vpn-gateway"></a>Línea de base de seguridad de Azure para VPN Gateway

Esta línea de base de seguridad aplica las instrucciones de [Azure Security Benchmark, versión 1.0](../security/benchmarks/overview-v1.md) a VPN Gateway. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa por medio de los **controles de seguridad** definidos por Azure Security Benchmark y las directrices relacionadas aplicables a VPN Gateway.

> [!NOTE]
> Se han excluido los **controles** que no son aplicables a VPN Gateway o cuya responsabilidad es de Microsoft. Para ver cómo VPN Gateway se asigna por completo a Azure Security Benchmark, consulte el **[archivo de asignación de base de referencia de seguridad de VPN Gateway completo](https://github.com/MicrosoftDocs/SecurityBenchmarks/raw/master/Azure%20Offer%20Security%20Baselines/1.1/vpn-gateway-security-baseline-v1.1.xlsx)** .

## <a name="network-security"></a>Seguridad de redes

*Para más información, consulte [Azure Security Benchmark: seguridad de red](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: cuando trabaje con subredes de VPN Gateway, evite asociar un grupo de seguridad de red (NSG) a la subred de la puerta de enlace. La asociación de un grupo de seguridad de red a esta subred puede causar que la puerta de enlace de VPN deje de funcionar como cabría esperar.  Sin embargo, habilite grupos de seguridad de red para otras subredes que no sean de VPN Gateway de su instancia de Virtual Network.

- [Creación de una red virtual](../virtual-network/quick-create-portal.md) 

- [Creación de un NSG con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md) 

- [Creación de una instancia de VPN Gateway basada en rutas mediante Azure Portal](/azure/vpn-gateway/create-routebased-vpn-gateway-portal)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Network**:

[!INCLUDE [Resource Policy for Microsoft.Network 1.1](../../includes/policy/standards/asb/rp-controls/microsoft.network-1-1.md)]

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes e interfaces de red

**Guía**: Use Azure Security Center y siga las recomendaciones de protección de redes para ayudar a proteger sus recursos de red en Azure. 

- [Descripción de la seguridad de red proporcionada por Azure Security Center](../security-center/security-center-network-recommendations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Network**:

[!INCLUDE [Resource Policy for Microsoft.Network 1.2](../../includes/policy/standards/asb/rp-controls/microsoft.network-1-2.md)]

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Guía**: habilite la captura de paquetes de VPN Gateway en la puerta de enlace o en una conexión específica según sus requisitos.

- [Configuración de captura de paquetes para VPN Gateways](packet-capture.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Network**:

[!INCLUDE [Resource Policy for Microsoft.Network 1.5](../../includes/policy/standards/asb/rp-controls/microsoft.network-1-5.md)]

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: Defina e implemente configuraciones de seguridad estándar para los recursos de red con Azure Policy.

También puede usar Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como plantillas de Azure Resource Manager, asignaciones de Azure RBAC y asignaciones de Azure Policy, en una única definición de un plano técnico. Puede aplicar el plano técnico a nuevas suscripciones o a suscripciones ya existentes y ajustar el control y la administración a través del control de versiones.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Ejemplos de Azure Policy para redes](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#network)

- [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: use el registro de actividad de Azure para supervisar las configuraciones de recursos y detectar cambios en los recursos de redes virtuales. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos críticos relacionados con su instancia de VPN Gateway.

- [Visualización y recuperación de eventos del registro de actividad de Azure](/azure/azure-monitor/essentials/activity-log-view)

- [Creación de alertas en Azure Monitor](../azure-monitor/alerts/alerts-activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Azure Security Benchmark: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: ingiera registros de actividad y diagnóstico a través de Azure Monitor para agregar datos de seguridad generados por recursos de red como sus recursos de VPN Gateway. Use Azure Monitor para realizar consultas y análisis de los datos de registro, y utilice cuentas de Azure Storage para el almacenamiento de archivos a largo plazo de estos registros. 

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

- [Configuración de alertas de eventos de registro de diagnóstico de VPN Gateway](vpn-gateway-howto-setup-alerts-virtual-network-gateway-log.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Recopilación de registros y métricas de plataforma con Azure Monitor](../azure-monitor/essentials/diagnostic-settings.md)

- [Introducción a Azure Monitor e integración con herramienta SIEM de terceros](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: habilite Configuración de diagnóstico en los recursos de VPN Gateway para poder acceder a los registros de auditoría, seguridad y diagnóstico. Los registros de actividad, que están disponibles automáticamente, incluyen el origen del evento, la fecha, el usuario, la marca de tiempo, las direcciones de origen y de destino, y otros elementos útiles. 

- [Recopilación de registros y métricas de plataforma con Azure Monitor](../azure-monitor/essentials/diagnostic-settings.md) 

- [Descripción del registro y de los distintos tipos de registro de Azure](../azure-monitor/essentials/platform-logs-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, establezca el período de retención del área de trabajo de Log Analytics de acuerdo con la normativa de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos y a largo plazo. 

- [Cambio del período de retención de datos en Log Analytics](https://docs.microsoft.com/azure/azure-monitor/logs/manage-cost-storage#change-the-data-retention-period) 

- [Configuración de la directiva de retención para los registros de la cuenta de Azure Storage](https://docs.microsoft.com/azure/storage/common/manage-storage-analytics-logs#configure-logging)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Guía**: Analice y supervise los registros en busca de comportamientos anómalos y revise los resultados con regularidad. Use Azure Monitor y un área de trabajo de Log Analytics para revisar los registros y realizar consultas en los datos de dichos registros. 

También puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros. 

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md) 

- [Introducción a las consultas de Log Analytics](/azure/azure-monitor/logs/get-started-portal) 

- [Procedimiento para realizar consultas personalizadas en Azure Monitor](../azure-monitor/logs/get-started-queries.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Instrucciones**: Use Azure Security Center con el área de trabajo de Log Analytics para la supervisión y alertas sobre actividades anómalas que se encuentran en los registros y eventos de seguridad.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel.

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Administración de alertas de seguridad en Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md)

- [Alertas sobre datos de registro de Log Analytics](../azure-monitor/alerts/tutorial-response.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: implemente una solución de terceros de Azure Marketplace para solucionar el registro DNS según los requisitos de su organización.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para más información, consulte [Azure Security Benchmark: identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: El control de acceso basado en roles de Azure (RBAC de Azure) permite administrar el acceso a los recursos de Azure a través de las asignaciones de roles. Puede asignar estos roles a usuarios, grupos de entidades de servicio e identidades administradas. Hay roles integrados predefinidos para determinados recursos, y estos roles se pueden inventariar o consultar mediante herramientas como la CLI de Azure, Azure PowerShell o el Azure Portal.

- [Obtención de un rol de directorio en Azure Active Directory (Azure AD) con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0&amp;preserve-view=true)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0&amp;preserve-view=true)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas.

También puede habilitar el acceso Just-in-Time mediante Azure Active Directory (Azure AD) Privileged Identity Management y Azure Resource Manager.

- [Más información acerca de Privileged Identity Management](/azure/active-directory/privileged-identity-management/)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4: Uso del inicio de sesión único (SSO) de Azure Active Directory

**Guía**: siempre que sea posible, use el inicio de sesión único de Azure Active Directory (Azure AD) en lugar de configurar credenciales independientes individuales por servicio. Use las recomendaciones de identidades y acceso de Azure Security Center.

- [Descripción del SSO con Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Habilite la autenticación multifactor de Azure Active Directory (Azure AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6: Uso de estaciones de trabajo seguras y administradas por Azure para realizar tareas administrativas

**Instrucciones**: use una estación de trabajo segura y administrada por Azure (también conocida como una estación de trabajo de acceso con privilegios o PAW) para las tareas administrativas que requieren privilegios elevados.

- [Descripción de las estaciones de trabajo seguras administradas por Azure](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Habilitación de la autenticación multifactor de Azure Active Directory (Azure AD)](../active-directory/authentication/howto-mfa-getstarted.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: use los informes de seguridad y la supervisión de Azure Active Directory (Azure AD) para detectar cuándo se producen actividades sospechosas o no seguras en el entorno. Use Azure Security Center para supervisar la actividad de identidad y acceso.

- [Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo](/azure/active-directory/reports-monitoring/concept-user-at-risk)

- [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: use ubicaciones con nombre de Azure Active Directory (Azure AD) para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

- [Configuración de ubicaciones con nombre de Azure AD](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios. 

- [Procedimiento para crear y configurar una instancia de Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Instrucciones**: Azure Active Directory (Azure AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso e identidades de Azure AD para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado. 

- [Descripción de los informes de Azure AD](/azure/active-directory/reports-monitoring/) 

- [Procedimiento para usar las revisiones de acceso e identidades de Azure AD](../active-directory/governance/access-reviews-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: tiene acceso a los orígenes de registro de eventos de riesgo, auditoría y actividad de inicio de sesión de Azure Active Directory (Azure AD), que permiten la integración en cualquier herramienta SIEM o de supervisión.

Para simplificar este proceso, cree una configuración de diagnóstico para las cuentas de usuario de Azure AD y envíe los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas deseadas en el área de trabajo de Log Analytics.

- [Integración de los registros de actividad de Azure en Azure Monitor](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Instrucciones**: use las características de protección de identidad de Azure Active Directory (Azure AD) para configurar respuestas automatizadas a las acciones sospechosas que se detecten relacionadas con las identidades de los usuarios. También puede hacer que Azure Sentinel ingiera los datos para investigarlos más.

- [Visualización de los inicios de sesión de riesgo de Azure AD](/azure/active-directory/reports-monitoring/concept-risky-sign-ins)

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-protection"></a>Protección de datos

*Para más información, consulte [Azure Security Benchmark: protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Guía**: las instancias de VPN Gateway son instancias de VM dedicadas para cada red virtual de cliente. Implemente el aislamiento mediante redes virtuales, suscripciones y grupos de administración independientes para dominios de seguridad individuales, como el tipo de entorno y el nivel de confidencialidad de los datos. Puede restringir el nivel de acceso a los recursos de Azure que necesitan sus aplicaciones y entornos empresariales. Puede controlar el acceso a los recursos de Azure mediante el control de acceso basado en rol de Azure (Azure RBAC).

- [Creación de suscripciones adicionales de Azure](/azure/billing/billing-create-subscription)

- [Creación de grupos de administración](/azure/governance/management-groups/create)

- [Creación y uso de etiquetas](/azure/azure-resource-manager/resource-group-using-tags)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Use una solución de terceros de Azure Marketplace en los perímetros de la red para supervisar la transferencia no autorizada de información confidencial y bloquear dichas transferencias al tiempo que alerta a los profesionales de seguridad de la información. 

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y protege a los clientes contra la pérdida y exposición de los datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos. 

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: paquetes de clientes cifrados de instancias de VPN Gateway entre las instancias de Azure VPN Gateway y los clientes de VPN (P2S) o los dispositivos de VPN (S2S) locales de los clientes. Las instancias de Azure VPN Gateway también admiten el cifrado entre redes virtuales.

Siga las recomendaciones de Azure Security Center para el cifrado en reposo y el cifrado en tránsito, para los recursos aplicables de la red virtual.

- [Acerca de los tipos de VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings#vpntype)

- [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec)

- [Acerca de los requisitos criptográficos y las puertas de enlace de VPN de Azure](vpn-gateway-about-compliance-crypto.md)

- [Configurar una directiva de IPsec o IKE para conexiones VPN de sitio a sitio o de red virtual a red virtual](vpn-gateway-ipsecikepolicy-rm-powershell.md)

- [Descripción del cifrado en tránsito con Azure](https://docs.microsoft.com/azure/security/fundamentals/encryption-overview#encryption-of-data-in-transit)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: use una herramienta de detección activa de terceros para identificar toda la información confidencial almacenada, procesada o transmitida por los sistemas tecnológicos, incluidos los de ubicación local o los que se encuentran en un proveedor de servicios remoto, y actualizar el inventario de información confidencial de la organización.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6: Uso del control de acceso basado en rol para controlar el acceso a los recursos

**Guía**: use el control de acceso basado en rol de Azure (Azure RBAC) para los datos y recursos; si no, use métodos de control de acceso específicos del servicio. Elija roles integrados como Propietario, Colaborador o Colaborador de red y asigne el rol al ámbito adecuado. Asigne permisos específicos para un subconjunto de funcionalidades de red virtual creando un rol personalizado y asignando a este los permisos específicos necesarios para redes virtuales, subredes, instancias de VPN Gateway, interfaces de red, grupos de seguridad de red y tablas de rutas.

- [Configuración de Azure RBAC](../role-based-access-control/role-assignments-portal.md)

- [Planear redes virtuales](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm#permissions)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: configure las alertas de Azure Monitor a fin de desencadenar los registros de actividad de Azure para cuando se produzcan cambios en recursos de Azure críticos como las instancias de VPN Gateway. 

- [Configuración de alertas en métricas de VPN Gateway](vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric.md)

- [Configuración de alertas de eventos de registro de diagnóstico de VPN Gateway](vpn-gateway-howto-setup-alerts-virtual-network-gateway-log.md)

- [Creación de alertas para los eventos del registro de actividad de Azure](/azure/azure-monitor/platform/alerts-activity-log)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Azure Security Benchmark: administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: Use un programa de puntuación de riesgos común (por ejemplo, Common Vulnerability Scoring System) o la clasificación de riesgos predeterminada proporcionada por su herramienta de análisis de terceros.

- [Publicación de NIST: sistema de puntuación común de vulnerabilidades](https://www.nist.gov/publications/common-vulnerability-scoring-system)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para más información, consulte [Azure Security Benchmark: inventario y administración de recursos](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: use Azure Resource Graph para consultar y detectar todos los recursos relacionados con sus instancias de VPN Gateway dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todos los recursos de las suscripciones. Además, también puede usar la CLI de Azure para enumerar los recursos de VPN Gateway.

- [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

- [CLI de Azure para VPN Gateway](/cli/azure/network/vnet-gateway)

- [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription)

- [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: aplique etiquetas a los recursos de Azure Gateway para organizarlos de forma lógica según una taxonomía definida.

- [Creación y uso de etiquetas](/azure/azure-resource-manager/resource-group-using-tags)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos de VPN Gateway. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna. 

- [Creación de suscripciones adicionales de Azure](/azure/billing/billing-create-subscription) 

- [Creación de grupos de administración](/azure/governance/management-groups/create) 

- [Creación y uso de etiquetas](/azure/azure-resource-manager/resource-group-using-tags)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: Cree un inventario de los recursos de Azure aprobados y el software aprobado para los recursos de proceso según las necesidades de la organización.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Además, use Azure Resource Graph para consultar y detectar recursos en las suscripciones. 

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

- [Integraciones de ejemplo de Azure Policy para la red virtual](/azure/virtual-network/policy-samples)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: los clientes pueden evitar el uso o la creación de recursos asignando definiciones de Azure Policy de acuerdo con los requisitos de seguridad de la organización. Sin embargo, debe implementar su propio proceso para quitar recursos no aprobados o no autorizados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#general)

- [Integraciones de ejemplo de Azure Policy para la red virtual](/azure/virtual-network/policy-samples)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Guía**: use el acceso condicional de Azure Active Directory (Azure AD) para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure".

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Azure Security Benchmark: configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: use alias de Azure Policy para crear directivas personalizadas con el fin de auditar o aplicar la configuración de los recursos de red de Azure, incluyendo VPN Gateway. También puede usar definiciones de Azure Policy integradas.

Azure Resource Manager tiene la capacidad de exportar la plantilla en formato de notación de objetos JavaScript (JSON), que se debe revisar para asegurarse de que las configuraciones cumplan o superen los requisitos de seguridad de la organización.

También puede usar las recomendaciones de Azure Security Center como línea de base de configuración segura para los recursos de Azure.

- [Visualización de los alias de Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-4.8.0&amp;preserve-view=true)

- [Tutorial: Creación y administración de directivas para aplicar el cumplimiento](../governance/policy/tutorials/create-and-manage.md)

- [Integraciones de ejemplo de Azure Policy para la red virtual](/azure/virtual-network/policy-samples)

- [Exportación de uno y varios recursos a una plantilla en Azure Portal](../azure-resource-manager/templates/export-template-portal.md)

- [Guía de referencia sobre las recomendaciones de seguridad](../security-center/recommendations-reference.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: use las plantillas de Azure Resource Manager y las asignaciones de Azure Policy para configurar de forma segura los recursos de Azure asociados a VPN Gateway y recursos relacionados. Las plantillas de Azure Resource Manager son archivos basados en JSON que se usan para implementar la máquina virtual junto con los recursos de Azure, y las plantillas personalizadas deberán mantenerse. Microsoft realiza el mantenimiento en las plantillas base.  Utilice los modos [deny] y [deploy if not exist] de Azure Policy para aplicar una configuración segura en los recursos de Azure.

- [Información sobre la creación de plantillas de Azure Resource Manager](../virtual-machines/windows/ps-template.md)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

- [Ejemplos de plantilla de Azure Resource Manager para una red virtual](../virtual-network/template-samples.md)

- [Integraciones de ejemplo de Azure Policy para la red virtual](/azure/virtual-network/policy-samples)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Use Azure DevOps para almacenar y administrar de forma segura el código, como definiciones de Azure Policy personalizadas, plantillas de Azure Resource Manager y scripts de Desired State Configuration. Para acceder a los recursos que administra en Azure DevOps, puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integran con Azure DevOps, o Azure AD si se integran con TFS.

- [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops&amp;preserve-view=true)

- [Acerca de los permisos y los grupos en Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: defina e implemente configuraciones de seguridad estándar para los recursos de Azure con Azure Policy. Use alias de Azure Policy para crear directivas personalizadas con el fin de auditar o aplicar las configuraciones de los recursos de Azure. También puede usar definiciones de directivas integradas relacionadas con recursos concretos. Además, puede usar Azure Automation para implementar los cambios de configuración. 

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Uso de alias](https://docs.microsoft.com/azure/governance/policy/concepts/definition-structure#aliases)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: asigne definiciones de Azure Policy para medir las configuraciones de recursos relacionadas con los recursos de VPN Gateway. Use la información de Azure Policy para auditar configuraciones de recursos y alertar sobre cambios de configuración críticos.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Integraciones de ejemplo de Azure Policy para la red virtual](/azure/virtual-network/policy-samples)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: el servicio VPN Gateway almacena internamente y transmite claves precompartidas y certificados del cliente cifrados. Los clientes deben proteger las claves precompartidas o los certificados en sus propios sistemas.

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: VPN de punto a sitio de Azure admite tres métodos de autenticación:

- Basada en certificados.
- RADIUS
- Azure Active Directory (Azure AD)

Se recomienda Azure AD porque le permite aprovechar las identidades administradas.

- [Configuración de un inquilino](openvpn-azure-ad-tenant.md)

- [Configuración de un inquilino con varias aplicaciones cliente](openvpn-azure-ad-tenant-multi-app.md)

- [Configuración de la autenticación multifactor](openvpn-azure-ad-mfa.md)

- [Configuración de un cliente VPN](openvpn-azure-ad-client.md)

- [Configuración de las identidades administradas](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault. 

- [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Azure Security Benchmark: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: use Azure Resource Manager para implementar recursos de VPN Gateway. Azure Resource Manager proporciona la capacidad de exportar plantillas, que se pueden usar como copias de seguridad para restaurar los recursos de VPN Gateway. Use Azure Automation para llamar a la API de exportación de plantillas de Azure Resource Manager de forma periódica.

- [Introducción sobre Azure Resource Manager](../azure-resource-manager/management/overview.md)

- [Ejemplos de plantilla de Azure Resource Manager para una red virtual](../virtual-network/template-samples.md)

- [Grupos de recursos: Exportar plantilla](/rest/api/resources/resourcegroups/exporttemplate)

- [Introducción a Azure Automation](../automation/automation-intro.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: use Azure Resource Manager para implementar recursos de VPN Gateway. Azure Resource Manager proporciona la capacidad de exportar plantillas, que se pueden usar como copias de seguridad para restaurar VPN Gateway y los recursos relacionados. Use Azure Automation para llamar a la API de exportación de plantillas de Azure Resource Manager de forma periódica.

- [Exportación de uno y varios recursos a una plantilla en Azure Portal](../azure-resource-manager/templates/export-template-portal.md)

- [Grupos de recursos: Exportar plantilla](/rest/api/resources/resourcegroups/exporttemplate)

- [Introducción a Azure Automation](../automation/automation-intro.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: en caso necesario, asegúrese de poder realizar periódicamente la implementación de plantillas de Azure Resource Manager de forma habitual en una suscripción aislada.

- [Implementación de recursos con Azure Portal y plantillas de Resource Manager](../azure-resource-manager/templates/deploy-portal.md)

- [Restauración de las claves del almacén de claves en Azure](https://docs.microsoft.com/powershell/module/az.keyvault/restore-azkeyvaultkey?view=azps-4.8.0&amp;preserve-view=true)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: Use Azure DevOps para almacenar y administrar de forma segura el código, como definiciones de Azure Policy personalizadas y plantillas de Azure Resource Manager. Para proteger los recursos que administra en Azure DevOps, puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integran con Azure DevOps.

- [Almacenamiento de código en Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops&amp;preserve-view=true)

- [Acerca de los permisos y los grupos en Azure DevOps](/azure/devops/organizations/security/about-permissions)

- [Eliminación temporal de blobs de Azure Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal)

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

**Instrucciones**: Azure Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en la métrica utilizados para emitir la alerta, así como en el nivel de confianza de que se produjo un intento malintencionado detrás de la actividad que provocó la alerta.

Adicionalmente, marque las suscripciones con etiquetas y cree un sistema de nomenclatura para identificar y clasificar los recursos de Azure, especialmente los que procesan datos confidenciales.  Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente. 

- [Alertas de seguridad en el Centro de seguridad de Azure](../security-center/security-center-alerts-overview.md) 

- [Uso de etiquetas para organizar los recursos de Azure](/azure/azure-resource-manager/resource-group-using-tags)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Instrucciones**: Realice ejercicios para probar las funcionalidades de respuesta a los incidentes de los sistemas periódicamente; así, ayudará a proteger los recursos de Azure. Identifique puntos débiles y brechas y después revise el plan de respuesta según sea necesario. 

- [Publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas. 

- [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Instrucciones**: exporte las alertas y recomendaciones de Azure Security Center y mediante la característica de exportación continua para ayudar a identificar riesgos en los recursos de Azure. La exportación continua permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para enviar las alertas a Azure Sentinel. 

- [Configuración de la exportación continua](../security-center/continuous-export.md) 

- [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: use la característica de automatización de flujos de trabajo de Azure Security Center para desencadenar automáticamente las alertas y recomendaciones de seguridad con el fin de proteger los recursos de Azure. 

- [Configuración de la automatización de flujos de trabajo en Security Center](../security-center/workflow-automation.md)

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
