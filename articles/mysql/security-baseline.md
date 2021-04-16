---
title: Línea de base de seguridad de Azure para Azure Database for MySQL
description: La línea de base de seguridad de Azure Database for MySQL proporciona instrucciones sobre procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: mysql
ms.topic: conceptual
ms.date: 03/29/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 62501d6174b9ef8806de0bbdae0dfedfe70da03a
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105968083"
---
# <a name="azure-security-baseline-for-azure-database-for-mysql"></a>Línea de base de seguridad de Azure para Azure Database for MySQL

Esta línea de base de seguridad aplica las instrucciones de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview-v1.md) a Azure Database for MySQL. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa por medio de los **controles de seguridad** definidos por Azure Security Benchmark y la guía relacionada aplicable a Azure Database for MySQL. Se han excluido los **controles** que no son aplicables a Azure Database for MySQL o cuya responsabilidad es de Microsoft.

Para ver cómo Azure Database for MySQL se asigna por completo a Azure Security Benchmark, consulte el [archivo completo de asignación de línea de base de seguridad de Azure Database for MySQL](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Seguridad de redes

*Para más información, consulte [Azure Security Benchmark: seguridad de red](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: Configuración de Private Link para Azure Database for MySQL con puntos de conexión privados. Private Link permite conectarse a varios servicios PaaS en Azure mediante un punto de conexión privado. En esencia, Azure Private Link incorpora los servicios de Azure dentro de su red virtual privada (VNet). El tráfico entre la red virtual y la instancia de MySQL viaja por la red troncal de Microsoft.

Como alternativa, puede usar puntos de conexión de servicio de red virtual para proteger y limitar el acceso de red a las implementaciones de Azure Database for MySQL. Las reglas de red virtual son una característica de firewall que controla si el servidor de Azure Database for MySQL acepta las comunicaciones que se envían desde subredes específicas en redes virtuales.

También puede proteger su servidor de Azure Database for MySQL con reglas de firewall. El firewall de servidor impide todo acceso al servidor de bases de datos, excepto a aquellos equipos a los que haya concedido permiso. Para configurar su firewall, cree reglas de firewall que especifiquen intervalos de direcciones IP aceptables. Puede crear reglas de firewall a nivel de servidor.

- [Configuración de Private Link para Azure Database for MySQL](howto-configure-privatelink-portal.md)

- [Creación y administración de reglas y puntos de conexión de servicio de red virtual de Azure Database for MySQL](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview)

- [Configuración de reglas de firewall para Azure Database for MySQL](howto-manage-firewall-using-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.DBforMySQL**:

[!INCLUDE [Resource Policy for Microsoft.DBforMySQL 1.1](../../includes/policy/standards/asb/rp-controls/microsoft.dbformysql-1-1.md)]

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes e interfaces de red

**Guía**: Cuando la instancia de Azure Database for MySQL está protegida en un punto de conexión privado, puede implementar máquinas virtuales en la misma red virtual. Puede usar un grupo de seguridad de red (NSG) para reducir el riesgo de la filtración de datos. Habilite los registros de flujo de NSG y envíe registros a una cuenta de almacenamiento para la auditoría del tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

- [Configuración de Private Link para Azure Database for MySQL](howto-configure-privatelink-portal.md)

- [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Denegación de las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: Use Advanced Threat Protection para Azure Database for MySQL. Advanced Threat Protection detecta actividades anómalas que indican intentos poco habituales y posiblemente dañinos de acceder a las bases de datos o aprovecharse de ellas.

Habilite el estándar de DDoS Protection en las redes virtuales asociadas a las instancias de Azure Database for MySQL a fin de protegerse de los ataques DDoS. Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

- [Configuración de Advanced Threat Protection para Azure Database for MySQL](howto-database-threat-protection-portal.md)

- [Configuración de la protección contra DDoS](../ddos-protection/manage-ddos-protection.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Guía**: Cuando la instancia de Azure Database for MySQL está protegida en un punto de conexión privado, puede implementar máquinas virtuales en la misma red virtual. A continuación, puede configurar un grupo de seguridad de red (NSG) para reducir el riesgo de la filtración de datos. Habilite los registros de flujo de NSG y envíe registros a una cuenta de almacenamiento para la auditoría del tráfico. También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

- [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implementación de sistemas de prevención de intrusiones y de detección de intrusiones (IDS/IPS) basados en la red.

**Guía**: Use Advanced Threat Protection para Azure Database for MySQL. Advanced Threat Protection detecta actividades anómalas que indican intentos poco habituales y posiblemente dañinos de acceder a las bases de datos o aprovecharse de ellas.

- [Configuración de Advanced Threat Protection para Azure Database for MySQL](howto-database-threat-protection-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: En el caso de los recursos que necesitan acceso a las instancias de Azure Database for MySQL, use las etiquetas de servicio de red virtual con el fin de definir controles de acceso a la red en grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, SQL.WestUs) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

Nota: Azure Database for MySQL usa las etiquetas de servicio "Microsoft.Sql".

- [Más información sobre el uso de etiquetas de servicio](../virtual-network/service-tags-overview.md)

- [Descripción del uso de etiquetas de servicio para Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/concepts-data-access-and-security-vnet#terminology-and-description)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: Defina e implemente configuraciones de seguridad estándar para los recursos y la configuración de red asociados a las instancias de Azure Database for MySQL con Azure Policy. Use alias de Azure Policy en los espacios de nombres "Microsoft.DBforMySQL" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las instancias de Azure Database for MySQL. También puede usar las definiciones de directiva integradas relacionadas con las redes o las instancias de Azure Database for MySQL, como:

- Se debe habilitar DDoS Protection estándar

- La aplicación de la conexión SSL debe estar habilitada para los servidores de bases de datos MySQL

Para más información, consulte las siguientes referencias:

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Ejemplos de Azure Policy para redes](/azure/governance/policy/samples/)

- [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: Use etiquetas para los recursos relacionados con la seguridad de red y el flujo de tráfico de las instancias de Azure Database for MySQL a fin de proporcionar metadatos y una organización lógica.

Use cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como **Requerir etiqueta y su valor**, para que todos los recursos se creen con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: Use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en dichos recursos relacionados con las instancias de Azure Database for MySQL. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

- [Visualización y recuperación de eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/essentials/activity-log#view-the-activity-log)

- [Creación de alertas en Azure Monitor](../azure-monitor/alerts/alerts-activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Azure Security Benchmark: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Habilite la configuración de diagnóstico y los registros de servidor y de ingesta para agregar los datos de seguridad generados por las instancias de Azure Database for MySQL. En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y utilice cuentas de Azure Storage para el almacenamiento de archivos a largo plazo. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

- [Descripción de los registros de servidor en Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/concepts-monitoring#server-logs)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Habilite Configuración de diagnóstico en las instancias de Azure Database for MySQL para poder acceder a los registros de auditoría, consultas lentas y métricas de MySQL. Asegúrese de habilitar específicamente el registro de auditoría de MySQL. Los registros de actividad, que están disponibles automáticamente, incluyen el origen del evento, la fecha, el usuario, la marca de tiempo, las direcciones de origen y de destino, y otros elementos útiles. También puede habilitar la configuración de diagnóstico del registro de actividad de Azure y enviar los registros a la misma área de trabajo de Log Analytics o cuenta de almacenamiento.

- [Descripción de los registros de servidor en Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/concepts-monitoring#server-logs)

- [Configuración y acceso a los registros de consultas lentas de Azure Database for MySQL](howto-configure-server-logs-in-portal.md)

- [Configuración y acceso a los registros de auditoría de Azure Database for MySQL](howto-configure-audit-logs-portal.md)

- [Configuración de las opciones de diagnóstico para el registro de actividad de Azure](../azure-monitor/essentials/activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: En Azure Monitor, en el área de trabajo de Log Analytics que se usa para almacenar los registros de Azure Database for MySQL, establezca el período de retención de acuerdo con las normas de cumplimiento de su organización. Use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

- [Configuración de parámetros de retención de registros de áreas de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/logs/manage-cost-storage#change-the-data-retention-period)

- [Almacenamiento de registros de recursos en una cuenta de Azure Storage](https://docs.microsoft.com/azure/azure-monitor/essentials/resource-logs#send-to-azure-storage)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Guía**: Analice y supervise los registros de las instancias de Azure Database for MySQL para comprobar si tienen comportamientos anómalos. Use Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos de registro. Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Para más información sobre Log Analytics, consulte ](../azure-monitor/logs/log-analytics-tutorial.md).

- [Procedimiento para realizar consultas personalizadas en Azure Monitor](../azure-monitor/logs/get-started-queries.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Instrucciones**: Habilitación de Advanced Threat Protection para Azure Database for MySQL. Advanced Threat Protection detecta actividades anómalas que indican intentos poco habituales y posiblemente dañinos de acceder a las bases de datos o aprovecharse de ellas.

Además, puede habilitar los registros de servidor y la configuración de diagnóstico para MySQL y enviar registros a un área de trabajo de Log Analytics. Incorpore el área de trabajo de Log Analytics a Azure Sentinel, ya que proporciona una solución de respuesta automatizada de orquestación de seguridad (SOAR). Esto permite crear cuadernos de estrategias (soluciones automatizadas) y usarlos para corregir problemas de seguridad.

- [Habilitación de Advanced Threat Protection para Azure Database for MySQL (versión preliminar)](howto-database-threat-protection-portal.md)

- [Descripción de los registros de servidor en Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/concepts-monitoring#server-logs)

- [Configuración y acceso a los registros de consultas lentas de Azure Database for MySQL](howto-configure-server-logs-in-portal.md)

- [Configuración y acceso a los registros de auditoría de Azure Database for MySQL](howto-configure-audit-logs-portal.md)

- [Configuración de las opciones de diagnóstico para el registro de actividad de Azure](../azure-monitor/essentials/activity-log.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para más información, consulte [Azure Security Benchmark: identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Instrucciones**: Mantenga un inventario de las cuentas de usuario que tienen acceso administrativo al plano de administración (por ejemplo, Azure Portal) de las instancias de Azure Database for MySQL. Así mismo, mantenga un inventario de las cuentas administrativas que tienen acceso al plano de datos (dentro de la propia base de datos) de las instancias de Azure Database for MySQL. (Al crear el servidor de MySQL, se deben proporcionar las credenciales de un usuario administrador. Este administrador se puede usar para crear más usuarios de MySQL).

Azure Database for MySQL no admite el control de acceso basado en roles integrado, pero se pueden crear roles personalizados basados en opciones específicas del proveedor de recursos.

- [Roles personalizados de Azure](../role-based-access-control/custom-roles.md) 

- [Descripción de las operaciones del proveedor de recursos de Azure Database for MySQL](https://docs.microsoft.com/azure/role-based-access-control/resource-provider-operations#microsoftdbformysql)

- [Descripción de la administración de acceso de Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/concepts-security#access-management)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: Azure Active Directory (Azure AD) no tiene el concepto de contraseñas predeterminadas.

Tras la creación del recurso de Azure Database for MySQL, Azure fuerza la creación de un usuario administrativo con una contraseña segura. Sin embargo, una vez creada la instancia de MySQL, puede usar la primera cuenta de administrador de servidor que creó a fin de crear usuarios adicionales y concederles acceso administrativo. Al crear estas cuentas, asegúrese de configurar una contraseña segura distinta para cada una.

- [Creación de cuentas adicionales en Azure Database for MySQL](howto-create-users.md)

- [Actualización de la contraseña del administrador](https://docs.microsoft.com/azure/mysql/howto-create-manage-server-portal#update-admin-password)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Instrucciones**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas que tengan acceso a las instancias de Azure Database for MySQL. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

- [Descripción de identidad y acceso en Azure Security Center](../security-center/security-center-identity-access.md)

- [Descripción de cómo crear usuarios administradores en Azure Database for MySQL](howto-create-users.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4: Uso del inicio de sesión único (SSO) de Azure Active Directory

**Guía**: es posible iniciar sesión en Azure Database for MySQL mediante el uso de un nombre de usuario y una contraseña configurados directamente en la base de datos, así como con el uso de una identidad de Azure Active Directory (Azure AD) y un token de Azure AD para la conexión. Al usar un token de Azure AD, se admiten varios métodos, como un usuario, un grupo o una aplicación de Azure AD que se conecten a la base de datos.

De forma independiente, el acceso al plano de control de MySQL está disponible a través de la API REST y es compatible con SSO. Para autenticarse, establezca el encabezado de autorización de las solicitudes en JSON Web Token, que se obtiene de Azure AD.

- [Uso de Azure AD para la autenticación con Azure Database for MySQL](howto-configure-sign-in-azure-ad-authentication.md)

- [Descripción de la API REST de Azure Database for MySQL](/rest/api/mysql/)

- [Descripción del SSO con Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: habilite la autenticación multifactor de Azure Active Directory (Azure AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center. Al utilizar tokens de Azure AD para iniciar sesión en la base de datos, esto le permite requerir la autenticación multifactor para inicios de sesión de base de datos.

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Uso de Azure AD para la autenticación con Azure Database for MySQL](howto-configure-sign-in-azure-ad-authentication.md)

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

**Guía**: Habilite Advanced Threat Protection para Azure Database for MySQL con el fin de generar alertas para actividades sospechosas.

Así mismo, puede usar Azure Active Directory (Azure AD) Privileged Identity Management (PIM) para la generación de registros y alertas cuando se produzca actividad sospechosa o insegura en el entorno.

Use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

- [Configuración de Advanced Threat Protection para Azure Database for MySQL](howto-database-threat-protection-portal.md)

- [Cómo implementar Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Información sobre las detecciones de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: Use ubicaciones con nombre de acceso condicional para permitir al portal y a Azure Resource Manager el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones.

- [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización. Azure AD protege los datos mediante un cifrado seguro para los datos en reposo y en tránsito. Azure AD también cifra con sal, convierte en hash y almacena de forma segura las credenciales de los usuarios.

Para iniciar sesión en Azure Database for MySQL, le recomendamos usar Azure AD y usar un token de Azure AD para conectarse. Al usar un token de Azure AD, se admiten varios métodos, como un usuario, un grupo o una aplicación de Azure AD que se conecten a la base de datos.

También se pueden usar credenciales de Azure AD para la administración a nivel de plano de administración (por ejemplo, Azure Portal) a fin de controlar las cuentas de administrador de MySQL.

- [Uso de Azure AD para la autenticación con Azure Database for MySQL](howto-configure-sign-in-azure-ad-authentication.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Instrucciones**: revise los registros de Azure Active Directory (Azure AD) a fin de detectar cuentas obsoletas, como aquellas con roles administrativos de Azure Database for MySQL. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficaz la pertenencia a grupos, el acceso a las aplicaciones empresariales que se pueden usar para tener acceso a Azure Database for MySQL y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica cada 90 días para asegurarse de que solo los usuarios adecuados tengan acceso continuado.

- [Descripción de los informes de Azure AD](/azure/active-directory/reports-monitoring/)

- [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: habilite la configuración de diagnóstico para Azure Database for MySQL y Azure Active Directory (Azure AD), y envíe todos los registros a un área de trabajo de Log Analytics. Configure las alertas que desee (como intentos de autenticación con errores) en Log Analytics.

- [Configuración y acceso a los registros de consultas lentas de Azure Database for MySQL](/Azure/mysql/howto-configure-server-logs-in-portal)

- [Configuración y acceso a los registros de auditoría de Azure Database for MySQL](/Azure/mysql/howto-configure-audit-logs-portal)

- [Integración de los registros de actividad de Azure en Azure Monitor](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12: Alerta de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Habilite Advanced Threat Protection para Azure Database for MySQL con el fin de generar alertas para actividades sospechosas.

Use las características Identity Protection y de detección de riesgos de Azure Active Directory (Azure AD) para configurar respuestas automatizadas a las acciones sospechosas detectadas. Puede habilitar las respuestas automatizadas a través de Azure Sentinel para implementar las respuestas de seguridad de su organización.

También puede hacer que Azure Sentinel ingiera los registros para investigarlos más.

- [Configuración de Advanced Threat Protection para Azure Database for MySQL](howto-database-threat-protection-portal.md)

- [¿Qué es Azure Active Directory Identity Protection?](../active-directory/identity-protection/overview-identity-protection.md)

- [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-protection"></a>Protección de datos

*Para más información, consulte [Azure Security Benchmark: protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: Use etiquetas para ayudar a realizar un seguimiento de las instancias de Azure Database for MySQL o los recursos relacionados que almacenan o procesan información confidencial.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Use una combinación de Private Link, puntos de conexión de servicio o reglas de firewall para aislar y limitar el acceso de red a las instancias de Azure Database for MySQL.

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Configuración de Private Link para Azure Database for MySQL](concepts-data-access-security-private-link.md)

- [Creación y administración de reglas y puntos de conexión de servicio de red virtual de Azure Database for MySQL](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview)

- [Configuración de reglas de firewall para Azure Database for MySQL](concepts-firewall-rules.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Al usar máquinas virtuales de Azure para tener acceso a instancias de Azure Database for MySQL, use Private Link, las configuraciones de red de MySQL, los grupos de seguridad de red y las etiquetas de servicio para mitigar la posibilidad de que se puedan filtrar los datos.

Microsoft administra la infraestructura subyacente de Azure Database for MySQL y ha implementado controles estrictos para evitar la pérdida o exposición de los datos de los clientes.

- [Mitigación de la filtración de datos para Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/concepts-data-access-security-private-link#data-exfiltration-prevention)

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Instrucciones**: Azure Database for MySQL permite conectar el servidor MySQL a las aplicaciones cliente con la Capa de sockets seguros (SSL). Aplicar conexiones SSL entre el servidor de base de datos y las aplicaciones cliente ayuda a proteger contra los ataques de tipo "man in the middle" mediante el cifrado del flujo de datos entre el servidor y la aplicación. En Azure Portal, asegúrese de que está habilitada la opción "Aplicar conexión SSL" para todas las instancias de Azure Database for MySQL de forma predeterminada.

Actualmente, las versiones de TLS compatibles con Azure Database for MySQL son TLS 1.0, TLS 1.1 y TLS 1.2.

- [Configuración del cifrado en tránsito para Azure Database for MySQL](concepts-ssl-connection-security.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.DBforMySQL**:

[!INCLUDE [Resource Policy for Microsoft.DBforMySQL 4.4](../../includes/policy/standards/asb/rp-controls/microsoft.dbformysql-4-4.md)]

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: Las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Database for MySQL. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6: Uso de RBAC de Azure para controlar el acceso a los recursos 

**Guía**: Use el control de acceso basado en rol de Azure (Azure RBAC) para controlar el acceso al plano de control de Azure Database for MySQL (por ejemplo, Azure Portal). Para el acceso al plano de datos (dentro de la base de datos), use consultas SQL para crear usuarios y configurar permisos de usuario. RBAC de Azure no afecta a los permisos de usuario de la base de datos.

- [Configuración de Azure RBAC](../role-based-access-control/role-assignments-portal.md)

- [Configuración del acceso de usuario con SQL para Azure Database for MySQL](howto-create-users.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en las instancias de producción de Azure Database for MySQL y otros recursos críticos o relacionados.

- [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/alerts/alerts-activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Azure Security Benchmark: administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: Siga las recomendaciones de Azure Security Center sobre la protección de Azure Database for MySQL y los recursos relacionados.

Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten Azure Database for MySQL.

- [Recomendaciones de Azure Security Center](../security-center/recommendations-reference.md)

- [Cobertura de características para los servicios de PaaS de Azure en Azure Security Center](../security-center/features-paas.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para más información, consulte [Azure Security Benchmark: inventario y administración de recursos](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Use Azure Resource Graph para consultar y detectar todos los recursos (incluidas las instancias de Azure Database for MySQL) dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

- [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription)

- [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: Aplique etiquetas a las instancias de Azure Database for MySQL y otros recursos relacionados que proporcionen metadatos para organizarlos lógicamente en una taxonomía.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar las instancias de Azure Database for MySQL y los recursos relacionados y hacer un seguimiento de ellos. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: No aplicable; esta recomendación está destinada a los recursos de proceso y a Azure en su conjunto.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Además, use Azure Resource Graph para consultar y detectar recursos en las suscripciones.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Para más información, consulte las siguientes referencias:

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](/azure/governance/policy/samples/index)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: Use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management. De este modo, se puede impedir la creación y los cambios en los recursos dentro de un entorno de alta seguridad, como instancias de Azure Database for MySQL que contienen información confidencial.

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Azure Security Benchmark: configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Defina e implemente configuraciones de seguridad estándar para las instancias de Azure Database for MySQL con Azure Policy. Use alias de Azure Policy en el espacio de nombres **Microsoft.DBforMySQL** para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de las instancias de Azure Database for MySQL. También puede usar las definiciones de directivas integradas relacionadas con las instancias de Azure Database for MySQL, como:

La aplicación de la conexión SSL debe estar habilitada para los servidores de bases de datos MySQL

- [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Use la directiva de Azure Policy [deny] y [deploy if not exist] para aplicar la configuración segura en los recursos de Azure.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Instrucciones**: Si usa definiciones personalizadas de Azure Policy para las instancias de Azure Database for MySQL y los recursos relacionados, use Azure Repos para almacenar y administrar el código de forma segura.

- [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [Documentación de Azure Repos](/azure/devops/repos/)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: Use alias de Azure Policy en el espacio de nombres "Microsoft.DBforMySQL" para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: Use alias de Azure Policy en el espacio de nombres **Microsoft.DBforMySQL** para crear directivas personalizadas con el fin de auditar y aplicar las configuraciones del sistema y enviar alertas sobre ellas. Use las directivas de Azure Policy [audit], [deny] e [deploy if not exist] para aplicar automáticamente las configuraciones de las instancias de Azure Database for MySQL y los recursos relacionados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Instrucciones**: En el caso de Azure Virtual Machines o las aplicaciones web que se ejecutan en Azure App Service y se usan para tener acceso a las instancias de Azure Database for MySQL, use Managed Service Identity junto con Azure Key Vault para simplificar y proteger la administración de secretos de Azure Database for MySQL. Asegúrese de que la eliminación temporal de Key Vault esté habilitada.

- [Integración con identidades administradas de Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

- [Creación de un almacén de claves](../key-vault/general/quick-create-portal.md)

- [Cómo proporcionar la autenticación de Key Vault con una identidad administrada](../key-vault/general/assign-access-policy-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Instrucciones**: la instancia de Azure Database for MySQL admite la autenticación de Azure Active Directory (Azure AD) para tener acceso a las bases de datos. Al crear la instancia de Azure Database for MySQL, se deben proporcionar las credenciales de un usuario administrador. Este administrador se puede usar para crear más usuarios de base de datos.

En el caso de Azure Virtual Machines o las aplicaciones web que se ejecutan en Azure App Service y se usan para tener acceso a las instancias de Azure Database for MySQL, use Managed Service Identity junto con Azure Key Vault para almacenar y recuperar las credenciales del servidor de Azure Database for MySQL. Asegúrese de que la eliminación temporal de Key Vault esté habilitada.

Use las identidades administradas para proporcionar a los servicios de Azure una identidad administrada automáticamente en Azure AD. Las identidades administradas le permiten autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluyendo Key Vault, sin necesidad de credenciales en el código.

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

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (por ejemplo, Azure Database for MySQL), pero no se ejecuta en el contenido del cliente.

Examine previamente el contenido que se carga en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, Azure Database for MySQL, etc. Microsoft no tiene acceso a los datos de estas instancias.

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Azure Security Benchmark: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Azure Database for MySQL realiza copias de seguridad de los archivos de datos y del registro de transacciones. En función del tamaño de almacenamiento máximo admitido, se realizan copias de seguridad completas y diferenciales (servidores de almacenamiento de 4 TB como máximo) o copias de seguridad de instantánea (servidores de almacenamiento de 16 TB como máximo). Estas copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención de copias de seguridad configurado. El período de retención predeterminado es siete días. Opcionalmente, puede configurarlo hasta 35 días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits.

- [Descripción de las copias de seguridad para Azure Database for MySQL](concepts-backup.md)

- [Descripción de la configuración inicial de Azure Database for MySQL](tutorial-design-database-using-portal.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.DBforMySQL**:

[!INCLUDE [Resource Policy for Microsoft.DBforMySQL 9.1](../../includes/policy/standards/asb/rp-controls/microsoft.dbformysql-9-1.md)]

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copias de seguridad de las claves administradas por el cliente

**Instrucciones**: Azure Database for MySQL crea automáticamente copias de seguridad del servidor y las almacena en el almacenamiento con redundancia local o geográfica en función de lo que elija el usuario. Las copias de seguridad pueden utilizarse para restaurar el servidor a un momento dado. Las copias de seguridad y las restauraciones son una parte esencial de cualquier estrategia de continuidad del negocio, ya que protegen los datos frente a daños o eliminaciones accidentales. 

Si usa Azure Key Vault para almacenar las credenciales de las instancias de Azure Database for MySQL, asegúrese de realizar copias de seguridad automatizadas de las claves. 

- [Descripción de las copias de seguridad para Azure Database for MySQL](howto-restore-server-portal.md) 

- [Procedimiento para realizar copias de seguridad de claves de Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.DBforMySQL**:

[!INCLUDE [Resource Policy for Microsoft.DBforMySQL 9.2](../../includes/policy/standards/asb/rp-controls/microsoft.dbformysql-9-2.md)]

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: En Azure Database for MySQL, al realizar una restauración se crea un nuevo servidor a partir de las copias de seguridad del servidor original. Hay dos tipos de restauración disponibles: Restauración a un momento dado y geográfica. La restauración a un momento dado está disponible con cualquier opción de redundancia de copia de seguridad y crea un nuevo servidor en la misma región que el servidor original. La restauración geográfica solo está disponible si ha configurado el servidor para el almacenamiento con redundancia geográfica y le permite restaurar el servidor en otra región.

El tiempo estimado de recuperación depende de varios factores, como el tamaño de la bases de datos, el tamaño del registro de transacciones, el ancho de banda de red y el número total de bases de datos que se están recuperando en la misma región al mismo tiempo. Normalmente, el tiempo de recuperación es inferior a 12 horas.

Pruebe periódicamente la restauración de las instancias de Azure Database for MySQL.

- [Descripción de la copia de seguridad y restauración en Azure Database for MySQL](concepts-backup.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Instrucciones**: Azure Database for MySQL realiza copias de seguridad completas, diferenciales y del registro de transacciones. Estas copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención de copias de seguridad configurado. El período de retención predeterminado es siete días. Opcionalmente, puede configurarlo hasta 35 días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits. Asegúrese de que la eliminación temporal de Key Vault esté habilitada.

- [Descripción de la copia de seguridad y restauración en Azure Database for MySQL](concepts-backup.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Azure Security Benchmark: respuesta ante incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

- [Configuración de las automatizaciones de flujos de trabajo en Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [El cliente también puede usar la guía de control de incidentes de seguridad de equipos de NIST como referencia para la creación de su propio plan de respuesta a incidentes](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Además, marque claramente las suscripciones (por ejemplo, producción, no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure.

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

**Guía**: Siga las reglas de compromiso de la prueba de penetración de Microsoft Cloud para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft.

- [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [Información general sobre Azure Security Benchmark V2](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
