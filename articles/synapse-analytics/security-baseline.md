---
title: Base de referencia de seguridad de Azure para Synapse Analytics
description: La línea de base de seguridad de Synapse Analytics proporciona instrucciones de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Bechmark.
author: msmbaldwin
ms.service: synapse-analytics
ms.subservice: security
ms.topic: conceptual
ms.date: 03/16/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 1a426a86ddb25733b8425c5887d2c0522f915e2d
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107587721"
---
# <a name="azure-security-baseline-for-synapse-analytics"></a>Base de referencia de seguridad de Azure para Synapse Analytics

Esta línea de base de seguridad aplica las instrucciones de [Azure Security Benchmark versión 1.0](../security/benchmarks/overview-v1.md) a Synapse Analytics. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure.
El contenido se agrupa con los **controles de seguridad** que define Azure Security Benchmark y las instrucciones relacionadas aplicables a Synapse Analytics. Se han excluido los **controles** no aplicables a Synapse Analytics.

 
Para ver cómo se asigna por completo Synapse Analytics a Azure Security Benchmark, consulte el [archivo completo de asignación de base de referencia de seguridad de Synapse Analytics](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Seguridad de redes

*Para más información, consulte [Azure Security Benchmark: seguridad de red](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Instrucciones**: Proteja su instancia de Azure SQL Server en una red virtual a través de Private Link. Private Link permite el acceso a los servicios PaaS de Azure a través de un punto de conexión privado en la red virtual. El tráfico entre la red virtual y el servicio viaja por la red troncal de Microsoft.

Como alternativa, al conectarse al grupo de Synapse SQL, limite el ámbito de la conexión saliente a la base de datos SQL mediante un grupo de seguridad de red. Deshabilite todo el tráfico de los servicios de Azure a la base de datos SQL mediante el punto de conexión público, para lo que debe establecer Allow Azure Services (Permitir servicios de Azure) en OFF (Desactivado). Asegúrese de que no se permiten direcciones IP públicas en las reglas de firewall.

- [¿Qué es Azure Private Link?](../private-link/private-link-overview.md)

- [Descripción de Private Link para Azure Synapse SQL](../azure-sql/database/private-endpoint-overview.md)

- [Creación de una red virtual](../virtual-network/quick-create-portal.md)

- [Creación de un NSG con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 1.1](../../includes/policy/standards/asb/rp-controls/microsoft.sql-1-1.md)]

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes e interfaces de red

**Guía**: Cuando se conecte al grupo de SQL dedicado después de habilitar los registros de flujo del grupo de seguridad de red (NSG), los registros se envían a una cuenta de Azure Storage para la auditoría de tráfico.

También puede enviar registros de flujo de grupo de seguridad de red a un área de trabajo de Log Analytics y usar el Análisis de tráfico para proporcionar información detallada sobre el flujo de tráfico en la nube de Azure. Algunas de las ventajas del Análisis de tráfico son la capacidad de visualizar la actividad de la red e identificar las zonas activas, identificar las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar configuraciones de red incorrectas.

- [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Descripción de la seguridad de red proporcionada por Azure Security Center](../security-center/security-center-network-recommendations.md)

- [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

- [Descripción de la seguridad de red proporcionada por Azure Security Center](../security-center/security-center-network-recommendations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Denegación de las comunicaciones con direcciones IP malintencionadas conocidas

**Instrucciones**: Use Advanced Threat Protection (ATP) para Azure Synapse SQL. ATP detecta actividades anómalas que indican intentos inusuales y potencialmente peligrosos de acceder a bases de datos, o de vulnerar su seguridad, y puede desencadenar distintas alertas, como "Posible inyección de código SQL" y "Acceso desde una ubicación inusual". Puede acceder a la característica ATP, que forma parte de la oferta de Advanced Data Security (ADS), así como administrarla, a través del portal central de ADS de SQL.

Habilite DDoS Protection estándar en las redes virtuales asociadas a Azure Synapse SQL para protegerse de los ataques por denegación de servicio distribuidos. Use la inteligencia sobre amenazas integrada de Azure Security Center para denegar las comunicaciones con direcciones IP malintencionadas conocidas o no utilizadas.

- [Descripción de ATP para Azure Synapse SQL](../azure-sql/database/threat-detection-overview.md)

- [Habilitación de Advanced Data Security para Azure SQL Database](../azure-sql/database/azure-defender-for-sql.md)

- [Información general de ADS](../azure-sql/database/azure-defender-for-sql.md)

- [Configuración de la protección contra DDoS](../ddos-protection/manage-ddos-protection.md)

- [Descripción de la inteligencia sobre amenazas integrada de Azure Security Center](../security-center/azure-defender.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Guía**: Cuando se conecte al grupo de SQL dedicado después de habilitar los registros de flujo del grupo de seguridad de red (NSG), envíe los registros a una cuenta de Azure Storage para la auditoría de tráfico. También puede enviar registros de flujo a un área de trabajo Log Analytics o transmitirlos a Event Hubs.  Si es necesario para investigar actividades anómalas, habilite la captura de paquetes de Network Watcher.

- [Habilitación de los registros de flujo de NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Habilitación de Network Watcher](../network-watcher/network-watcher-create.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implementación de sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Instrucciones**: Use Advanced Threat Protection (ATP) para Azure Synapse SQL. ATP detecta actividades anómalas que indican intentos inusuales y potencialmente peligrosos de acceder a bases de datos, o de vulnerar su seguridad, y puede desencadenar distintas alertas, como "Posible inyección de código SQL" y "Acceso desde una ubicación inusual". Puede acceder a la característica ATP, que forma parte de la oferta de Advanced Data Security (ADS), así como administrarla, a través del portal central de ADS de SQL. ATP también integra alertas con Azure Security Center.

- [Descripción de ATP para Azure Synapse SQL](../azure-sql/database/threat-detection-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: Puede usar etiquetas de servicio de red virtual para definir controles de acceso a la red en los grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, ApiManagement) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

Cuando se usa un punto de conexión de servicio para el grupo de SQL dedicado, se requiere una salida a las direcciones IP públicas de la base de datos de Azure SQL: los grupos de seguridad de red (NSG) deben estar abiertos en las direcciones IP de Azure SQL Database para permitir la conectividad. Puede hacerlo mediante el uso de etiquetas de servicio de grupos de seguridad de red para Azure SQL Database.

- [Descripción de las etiquetas de servicio con puntos de conexión de servicio para Azure SQL Database](https://docs.microsoft.com/azure/azure-sql/database/vnet-service-endpoint-rule-overview#limitations)

- [Descripción y uso de etiquetas de servicio](../virtual-network/service-tags-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: Defina e implemente configuraciones de seguridad de red para recursos relacionados con su grupo de SQL dedicado con Azure Policy. Puede usar el espacio de nombres "Microsoft.Sql" para detallar definiciones de directiva personalizadas o usar cualquiera de las definiciones de directiva integradas diseñadas para la protección de red de Azure SQL Database o el servidor. Un ejemplo de una directiva de seguridad de red integrada aplicable para Azure SQL Database Server sería: "SQL Server debe usar un punto de conexión de servicio de red virtual".

Use Azure Blueprints para simplificar las implementaciones de Azure a gran escala mediante el empaquetado de artefactos de entorno clave, como las plantillas de Azure Resource Manager, el control de acceso basado en rol de Azure (Azure RBAC) y las directivas, en una única definición de un plano técnico. Aplique fácilmente el plano técnico a nuevas suscripciones y entornos, y ajuste el control y la administración mediante el control de versiones.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de un plano técnico de Azure](../governance/blueprints/create-blueprint-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Instrucciones**: use etiquetas para grupos de seguridad de red (NSG) y otros recursos relacionados con la seguridad de red y el flujo de tráfico. En el caso de las reglas de NSG individuales, use el campo "Descripción" para especificar las necesidades empresariales o la duración (etc.) de las reglas que permiten que entre o salga el tráfico en una red.

Use cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor", para asegurarse de que todos los recursos se creen con etiquetas y para notificarle los recursos no etiquetados existentes.

Puede usar Azure PowerShell o la CLI de Azure para buscar o realizar acciones en los recursos en función de sus etiquetas.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: Use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en los recursos de red relacionados con el grupo de SQL dedicado. Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.

- [Visualización y recuperación de eventos del registro de actividad de Azure](https://docs.microsoft.com/azure/azure-monitor/essentials/activity-log#view-the-activity-log)

- [Creación de alertas en Azure Monitor](../azure-monitor/alerts/alerts-activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Azure Security Benchmark: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Se puede definir una directiva de auditoría para una base de datos específica o como directiva de servidor predeterminada en Azure (que hospeda Azure Synapse). Una directiva de servidor se aplica a todas las bases de datos recién creadas en el servidor.

Si la auditoría de servidor está habilitada, se aplica siempre a la base de datos. La base de datos se auditará, independientemente de la configuración de auditoría de la base de datos.

Al habilitar la auditoría, los resultados se pueden escribir en un registro de auditoría en la cuenta de Azure Storage, el área de trabajo de Log Analytics o Event Hubs.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

- [Configuración de la auditoría para los recursos de Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/auditing-overview#server-vs-database-level)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Habilite la auditoría en el nivel de servidor de Azure SQL para el grupo de SQL dedicado y elija una ubicación de almacenamiento para los registros de auditoría (Azure Storage, Log Analytics o Event Hubs).

La auditoría se puede habilitar tanto en el nivel de base de datos como en el servidor. Se recomienda habilitarla solo en el nivel de servidor, a menos que sea necesario configurar un receptor de datos independiente o una retención para una base de datos específica.

- [Habilitación de la auditoría de Azure SQL Database](../azure-sql/database/auditing-overview.md)

- [Habilitación de la auditoría del servidor](https://docs.microsoft.com/azure/azure-sql/database/auditing-overview#setup-auditing)

- [Diferencias de la directiva de auditoría de nivel de servidor frente la de nivel de base de datos](https://docs.microsoft.com/azure/azure-sql/database/auditing-overview#server-vs-database-level)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 2.3](../../includes/policy/standards/asb/rp-controls/microsoft.sql-2-3.md)]

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: Al almacenar los registros relacionados con el grupo de SQL dedicado en una cuenta de Storage, el área de trabajo de Log Analytics o Event Hubs, establezca el período de retención de registro según la normativa de cumplimiento de su organización.

- [Administración del ciclo de vida de Azure Blob Storage](../storage/blobs/storage-lifecycle-management-concepts.md)

- [Configuración de parámetros de retención de registros de un área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/logs/manage-cost-storage#change-the-data-retention-period)

- [Captura de eventos de streaming en Event Hubs](../event-hubs/event-hubs-capture-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 2.5](../../includes/policy/standards/asb/rp-controls/microsoft.sql-2-5.md)]

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Instrucciones**: analice y supervise los registros en busca de comportamientos anómalos y revise los resultados con regularidad. Use Advanced Threat Protection para Azure SQL Database junto con Azure Security Center para alertar sobre actividad inusual relacionada con su instancia de SQL Database. También puede configurar alertas basadas en valores de métricas o en entradas del registro de actividad de Azure relacionadas con las instancias de SQL Database.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel o en una herramienta SIEM de terceros.

- [Descripción de Advanced Threat Protection y alertas de Azure SQL Database](../azure-sql/database/threat-detection-overview.md)

- [Habilitación de Advanced Data Security para Azure SQL Database](../azure-sql/database/azure-defender-for-sql.md)

- [Configuración de alertas personalizadas para Azure SQL Database](../azure-sql/database/alerts-insights-configure-portal.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Use Advanced Threat Protection (ATP) para Azure SQL Database junto con de Azure Security Center para supervisar cualquier actividad anómala y alertar sobre ella. ATP forma parte de la oferta de seguridad de datos avanzada (ADS), y se puede acceder a esta y administrarla mediante la funcionalidad ADS central de SQL en el portal. ADS incluye una funcionalidad para detectar y clasificar datos confidenciales, buscar y mitigar los puntos vulnerables potenciales de una base de datos, así como para detectar actividades anómalas que puedan indicar una amenaza para su base de datos.

Como alternativa, puede habilitar e incorporar datos en Azure Sentinel.

- [Descripción de Advanced Threat Protection y alertas de Azure SQL Database](../azure-sql/database/threat-detection-overview.md)

- [Habilitación de Advanced Data Security para Azure SQL Database](../azure-sql/database/azure-defender-for-sql.md)

- [Administración de alertas de seguridad en Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 2.7](../../includes/policy/standards/asb/rp-controls/microsoft.sql-2-7.md)]

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para más información, consulte [Azure Security Benchmark: identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: los usuarios se autentican con Azure Active Directory (Azure AD) o con la autenticación de SQL.

Al crear la primera implementación de Azure SQL, hay que especificar un inicio de sesión de administrador y una contraseña asociada a ese inicio de sesión. Esta cuenta administrativa se denomina "administrador del servidor". Para saber cuáles son las cuentas de administrador de una base de datos, abra Azure Portal y vaya a la pestaña Propiedades de su servidor o instancia administrada. También puede configurar una cuenta de administrador de Azure AD con permisos administrativos completos, lo que es necesario si quiere habilitar la autenticación de Azure AD.

En el caso de las operaciones de administración, use los roles integrados de Azure, que deben asignarse explícitamente. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc a fin de detectar cuentas que son miembros de grupos administrativos.

- [Autenticación de Azure SQL Database](https://docs.microsoft.com/azure/azure-sql/database/security-overview#authentication)

- [Creación de cuentas para usuarios que no son administradores](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#create-accounts-for-non-administrator-users)

- [Uso de una cuenta de Azure AD para la autenticación](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#create-additional-logins-and-users-having-administrative-permissions)

- [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

- [Cómo administrar inicios de sesión y cuentas de administrador existentes en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

- [Roles integrados en los recursos de Azure](../role-based-access-control/built-in-roles.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: Azure Active Directory (Azure AD) no tiene el concepto de contraseñas predeterminadas. Al aprovisionar un grupo de SQL dedicado, se recomienda optar por integrar la autenticación con Azure AD. Con este método de autenticación, el usuario envía un nombre de cuenta de usuario y solicita que el servicio use la información de credenciales almacenada en Azure AD.

- [Configuración y administración de la autenticación de Azure AD con Azure SQL](../azure-sql/database/authentication-aad-configure.md)

- [Descripción de la autenticación en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Instrucciones**: cree directivas y procedimientos en torno al uso de cuentas administrativas dedicadas. Use Administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas en las que se inicia sesión a través de Azure Active Directory (Azure AD).

Para saber cuáles son las cuentas de administrador de una base de datos, abra Azure Portal y vaya a la pestaña Propiedades de su servidor o instancia administrada.

- [Descripción de identidad y acceso en Azure Security Center](../security-center/security-center-identity-access.md)

- [Cómo administrar inicios de sesión y cuentas de administrador existentes en Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4: Uso del inicio de sesión único (SSO) de Azure Active Directory

**Guía**: Use un registro de aplicaciones de Azure (entidad de servicio) para recuperar un token que se pueda usar para interactuar con su almacén de datos en el plano de control (Azure Portal) a través de llamadas API.

- [Llamada a las API REST de Azure](/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

- [Registro de la aplicación cliente (entidad de servicio) con Azure Active Directory (Azure AD)](/rest/api/azure/#register-your-client-application-with-azure-ad)

- [Información de la API de REST de Azure Synapse SQL](sql-data-warehouse/sql-data-warehouse-manage-compute-rest-api.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: habilite la autenticación multifactor de Azure Active Directory (Azure AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

- [Explicación de la autenticación multifactor en Azure SQL](../azure-sql/database/authentication-mfa-ssms-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6: Uso de estaciones de trabajo seguras y administradas por Azure para realizar tareas administrativas

**Guía**: utilice estaciones de trabajo de acceso con privilegios (PAW) que tengan configurada la autenticación multifactor para iniciar sesión en los recursos de Azure y configurarlos.

- [Más información sobre las estaciones de trabajo con privilegios de acceso](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: Use los informes de seguridad de Azure Active Directory (Azure AD) para la generación de registros y alertas cuando se produzcan actividades sospechosas o no seguras en el entorno.

Use Advanced Threat Protection para Azure SQL Database junto con Azure Security Center para la detección y alerta de actividades anómalas que indican intentos inusuales y potencialmente dañinos de acceso o ataque a las bases de datos.

La auditoría de SQL Server le permite crear auditorías de servidor, que pueden contener especificaciones de auditoría de servidor para los eventos del nivel de servidor así como especificaciones de auditoría de base de datos para los eventos del nivel de base de datos. Los eventos auditados se pueden escribir en los registros de eventos o en los archivos de auditoría.

- [Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo](../active-directory/identity-protection/overview-identity-protection.md)

- [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](../security-center/security-center-identity-access.md)

- [Revisión de Advanced Threat Protection y posibles alertas](https://docs.microsoft.com/azure/azure-sql/database/threat-detection-overview#alerts)

- [Descripción de inicios de sesión y cuentas de usuario en Azure SQL](../azure-sql/database/logins-create-manage.md)

- [Descripción de la auditoría de SQL Server](/sql/relational-databases/security/auditing/sql-server-audit-database-engine)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Instrucciones**: use ubicaciones con nombre de acceso condicional para permitir al portal y a la Administración de recursos de Azure el acceso solo desde agrupaciones lógicas de intervalos de direcciones IP o países o regiones específicos.

- [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: cree un administrador de Azure Active Directory (Azure AD) para el servidor de Azure SQL Database en su grupo de SQL dedicado.

- [Configuración y administración de la autenticación de Azure AD con Azure SQL](../azure-sql/database/authentication-aad-configure.md)

- [Procedimiento para crear y configurar una instancia de Azure AD](../active-directory-domain-services/tutorial-create-instance.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 3.9](../../includes/policy/standards/asb/rp-controls/microsoft.sql-3-9.md)]

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Azure Active Directory (Azure AD) proporciona registros para ayudarle a descubrir cuentas obsoletas. Además, use las revisiones de acceso de Azure AD para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo los usuarios adecuados tengan acceso continuado.

Si se usa la autenticación de SQL, cree usuarios de base de datos independientes en la base de datos. Asegúrese de asignar a uno o varios usuarios de base de datos un rol de base de datos personalizado con permisos específicos adecuados para ese grupo de usuarios.

- [Uso de las revisiones de acceso](../active-directory/governance/access-reviews-overview.md)

- [Descripción de inicios de sesión y cuentas de usuario en Azure SQL](../azure-sql/database/logins-create-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: configure la autenticación de Azure Active Directory (Azure AD) con Azure SQL y habilite la configuración de diagnóstico para las cuentas de usuario de Azure AD, de forma que los registros de auditoría e inicio de sesión se envíen a un área de trabajo de Log Analytics. Configure las alertas deseadas en el área de trabajo de Log Analytics.

Si se usa la autenticación de SQL, cree usuarios de base de datos independientes en la base de datos. Asegúrese de asignar a uno o varios usuarios de base de datos un rol de base de datos personalizado con permisos específicos adecuados para ese grupo de usuarios.

- [Uso de las revisiones de acceso](../active-directory/governance/access-reviews-overview.md)

- [Configuración y administración de la autenticación de Azure AD con Azure SQL Database](../azure-sql/database/authentication-aad-configure.md)

- [Integración de los registros de actividad de Azure en Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Descripción de inicios de sesión y cuentas de usuario en Azure SQL](../azure-sql/database/logins-create-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12: Alerta de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Instrucciones**: Use las características de protección de identidad y detección de riesgo de Azure Active Directory (Azure AD) para configurar respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. Asimismo, incorpore e ingiera datos en Azure Sentinel para investigarlos más a fondo.

Si se usa la autenticación de SQL, cree usuarios de base de datos independientes en la base de datos. Asegúrese de asignar a uno o varios usuarios de base de datos un rol de base de datos personalizado con permisos específicos adecuados para ese grupo de usuarios.

- [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Incorporación de Azure Sentinel](../sentinel/connect-data-sources.md)

- [Descripción de inicios de sesión y cuentas de usuario en Azure SQL](../azure-sql/database/logins-create-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Guía**: En escenarios de soporte técnico en los que Microsoft necesita acceder a datos relacionados con Azure SQL Database de su grupo de SQL dedicado, la característica Caja de seguridad del cliente de Azure proporciona una interfaz para que pueda revisar y aprobar o rechazar las solicitudes de acceso a los datos del cliente.

- [Descripción de la Caja de seguridad del cliente](../security/fundamentals/customer-lockbox-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-protection"></a>Protección de datos

*Para más información, consulte [Azure Security Benchmark: protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: use etiquetas para ayudar a realizar el seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

La clasificación y detección de datos está integrada en Azure Synapse SQL. Ofrece funcionalidades avanzadas para detectar, clasificar, etiquetar e informar de los datos confidenciales de las bases de datos.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

- [Descripción de la clasificación y la detección de datos](../azure-sql/database/data-discovery-and-classification-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 4.1](../../includes/policy/standards/asb/rp-controls/microsoft.sql-4-1.md)]

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Instrucciones**: Implemente suscripciones y/o grupos de administración independientes para los entornos de desarrollo, prueba y producción. Los recursos deben separarse por red virtual o subred, etiquetarse adecuadamente y protegerse en un grupo de seguridad de red o Azure Firewall. Los recursos que almacenan o procesan datos confidenciales deben estar aislados. Use Private Link; implemente Azure SQL Server en una red virtual y conéctese de forma segura mediante Private Link.

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

- [Configuración de Private Link para Azure SQL Database](../azure-sql/database/private-endpoint-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: En el caso de cualquier instancia de Azure SQL Database de su grupo de SQL dedicado que almacene o procese información confidencial, marque la base de datos y los recursos relacionados como confidenciales mediante etiquetas. Configure Private Link junto con las etiquetas de servicio del grupo de seguridad de red (NSG) en las instancias de Azure SQL Database para evitar la filtración de información confidencial.

Además, Advanced Threat Protection para Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse detecta actividades anómalas que indican intentos inusuales y potencialmente peligrosos de acceder a las bases de datos o de vulnerar su seguridad.

En el caso de la plataforma subyacente administrada por Microsoft, Microsoft trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

- [Configuración de Private Link y los grupos de seguridad de red con el fin de evitar el filtrado de datos en las instancias de Azure SQL Database](../azure-sql/database/private-endpoint-overview.md)

- [Configuración de Advanced Threat Protection para Azure SQL Database](../azure-sql/database/threat-detection-overview.md)

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: Use la característica de clasificación y detección de datos de Azure Synapse SQL. La clasificación y detección de datos proporciona funcionalidades avanzadas integradas en Azure SQL Database para detectar, clasificar, etiquetar y proteger los datos confidenciales de las bases de datos.

La clasificación y detección de datos forma parte de la oferta de Advanced Data Security. Dicha oferta es un paquete unificado de funcionalidades avanzadas de seguridad de SQL. Se puede acceder a la clasificación y detección de datos, así como administrarla, desde el portal de ADS de SQL.

Además, puede configurar una directiva de enmascaramiento dinámico de datos (DDM) en Azure Portal. El motor de recomendaciones de DDM marca determinados campos de la base de datos como campos potencialmente confidenciales, que pueden ser buenos candidatos para el enmascaramiento.

- [Uso de la detección y la clasificación de datos para Azure SQL Server](../azure-sql/database/data-discovery-and-classification-overview.md)

- [Descripción del enmascaramiento dinámico de datos para Azure Synapse SQL](../azure-sql/database/dynamic-data-masking-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 4.5](../../includes/policy/standards/asb/rp-controls/microsoft.sql-4-5.md)]

### <a name="46-use-azure-rbac-access-control-to-control-access-to-resources"></a>4.6: Uso del control de acceso RBAC de Azure para controlar el acceso a los recursos 

**Guía**: Use el control de acceso basado en roles (Azure RBAC) de Azure para administrar el acceso a las instancias de Azure SQL Database en el grupo de SQL dedicado.

La autorización se controla por medio de las pertenencias a roles y los permisos de nivel de objeto de la base de datos de la cuenta de usuario. Como procedimiento recomendado, debe conceder a los usuarios los privilegios mínimos necesarios.

- [Integración de Azure SQL Server con Azure Active Directory (Azure AD) para la autenticación](../azure-sql/database/authentication-aad-overview.md)

- [Control del acceso en Azure SQL Server](../azure-sql/database/logins-create-manage.md)

- [Descripción de la autorización y la autenticación en Azure SQL](../azure-sql/database/logins-create-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Instrucciones**: El Cifrado de datos transparente (TDE) ayuda a proteger Azure Synapse SQL frente a amenazas de actividad sin conexión malintencionada mediante el cifrado de datos en reposo. También realiza cifrado y descifrado de la base de datos en tiempo real, copias de seguridad asociadas y archivos de registro de transacciones en reposo sin necesidad de efectuar cambios en la aplicación. En Azure, la configuración predeterminada de TDE es que la clave de cifrado está protegida mediante un certificado de servidor integrado. Asimismo, puede usar el cifrado TDE administrado por el cliente, que también se conoce como compatibilidad de Bring Your Own Key (BYOK) con TDE. En este escenario, el protector de TDE que cifra la clave de cifrado es una clave asimétrica administrada por el cliente, que se almacena en una instancia de Azure Key Vault que es propiedad del cliente y que este administra (un sistema de administración de claves externas basado en la nube de Azure), y que nunca sale del almacén de claves.

- [Descripción del cifrado de datos transparente administrado por el servicio](../azure-sql/database/transparent-data-encryption-tde-overview.md)

- [Descripción del cifrado de datos transparente administrado por el cliente](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-tde-overview#customer-managed-transparent-data-encryption---bring-your-own-key)

- [Cómo activar TDE con su propia clave](../azure-sql/database/transparent-data-encryption-byok-configure.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 4.8](../../includes/policy/standards/asb/rp-controls/microsoft.sql-4-8.md)]

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en las instancias de producción de grupos de Synapse SQL y otros recursos críticos o relacionados.

Además, puede configurar alertas para las bases de datos de su grupo de Synapse SQL mediante Azure Portal. Las alertas pueden enviarle un correo electrónico o llamar a un webhook cuando alguna métrica (por ejemplo, el tamaño de la base de datos o el uso de la CPU) alcanza el umbral.

- [Creación de alertas para los eventos del registro de actividad de Azure](../azure-monitor/alerts/alerts-activity-log.md)

- [Creación de alertas para Azure Synapse SQL](../azure-sql/database/alerts-insights-configure-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Azure Security Benchmark: administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: habilite Advanced Data Security y siga las recomendaciones de Azure Security Center sobre cómo realizar evaluaciones de vulnerabilidades en sus instancias de Azure SQL Database.

- [Ejecución de evaluaciones de vulnerabilidad en instancias de Azure SQL Database](../azure-sql/database/sql-vulnerability-assessment.md)

- [Habilitación de Advanced Data Security](../azure-sql/database/azure-defender-for-sql.md)

- [Implementación de las recomendaciones de evaluación de vulnerabilidades de Azure Security Center](../security-center/deploy-vulnerability-assessment-vm.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 5.1](../../includes/policy/standards/asb/rp-controls/microsoft.sql-5-1.md)]

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Guía**: La evaluación de vulnerabilidades es un servicio de detección integrado en Azure Synapse SQL. Este servicio emplea una base de conocimiento de reglas que marcan las vulnerabilidades de seguridad. Se resaltan las desviaciones de los procedimientos recomendados, como configuraciones inadecuadas, permisos excesivos y datos confidenciales desprotegidos.  Puede acceder a la evaluación de vulnerabilidades y administrarla a través del portal central de Advanced Data Security (ADS) de SQL.

- [Administración y exportación de exámenes de evaluación de vulnerabilidades en el portal de ADS de SQL](../azure-sql/database/sql-vulnerability-assessment.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: Use la clasificación de riesgo predeterminada (puntuación de seguridad) proporcionada por Azure Security Center.

La clasificación y detección de datos está integrada en Azure Synapse SQL. Ofrece funcionalidades avanzadas para detectar, clasificar, etiquetar e informar de los datos confidenciales de las bases de datos.

- [Descripción de la puntuación de seguridad de Azure Security Center](../security-center/secure-score-security-controls.md)

- [Descripción de la clasificación y la detección de datos](../azure-sql/database/data-discovery-and-classification-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 5.5](../../includes/policy/standards/asb/rp-controls/microsoft.sql-5-5.md)]

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para más información, consulte [Azure Security Benchmark: inventario y administración de recursos](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Use Azure Resource Graph para consultar y detectar todos los recursos relacionados con su grupo de SQL dedicado dentro de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Azure Resource Graph, se recomienda encarecidamente crear y usar los recursos de Azure Resource Manager que figuran a continuación.

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

**Instrucciones**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: Defina una lista de recursos de Azure aprobados relacionados con su grupo de SQL dedicado.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Use Azure Resource Graph para consultar o detectar recursos en las suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de consultas con Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Instrucciones**: use Azure Policy para establecer restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Use Azure Resource Graph para consultar o detectar recursos dentro de las suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#general)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure".

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Azure Security Benchmark: configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Use alias de Azure Policy en el espacio de nombres "Microsoft.Sql" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de recursos relacionada con el grupo de SQL dedicado. También puede usar definiciones de directiva integradas en servidores o bases de datos de Azure, como:

- Implementación de la detección de amenazas en servidores SQL Server.
- SQL Server debe usar un punto de conexión del servicio de red virtual

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

**Guía**: Si usa definiciones de personalizadas de Azure Policy, use Azure DevOps o Azure Repos para almacenar y administrar el código de forma segura.

- [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [Documentación de Azure Repos](/azure/devops/repos/)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: No aplicable; Azure Synapse SQL no tiene opciones de configuración de seguridad configurables.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: Use Azure Security Center para realizar exámenes de base de referencia de los recursos relacionados con su grupo de SQL dedicado.

- [Corrección de recomendaciones en Azure Security Center](../security-center/security-center-remediate-recommendations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: el Cifrado de datos transparente (TDE) con claves administradas por el cliente en Azure Key Vault permite cifrar la clave de cifrado de base de datos (DEK) generada automáticamente con una clave asimétrica administrada por el cliente que se conoce como protector de TDE. Esto también se conoce como compatibilidad de Bring Your Own Key (BYOK) para Cifrado de datos transparente. En el escenario BYOK, el protector de TDE se almacena en una instancia de Azure Key Vault administrada por el cliente, y de la que este es propietario. Asimismo, asegúrese de que la eliminación temporal está habilitada en Azure Key Vault.

- [Habilitación del cifrado TDE con una clave administrada por el cliente de Azure Key Vault](../azure-sql/database/transparent-data-encryption-byok-configure.md)

- [Habilitación de la eliminación temporal en Azure Key Vault](../key-vault/general/key-vault-recovery.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Instrucciones**: Use identidades administradas para proporcionar servicios de Azure con una identidad administrada automáticamente en Azure Active Directory (Azure AD). Identidades administradas le permite autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluido Azure Key Vault, sin necesidad de credenciales en el código.

- [Tutorial: Uso de una identidad administrada asignada por el sistema de una VM Windows para acceder a Azure SQL](../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-sql.md)

- [Configuración de las identidades administradas](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: implemente Credential Scanner para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

- [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Azure Security Benchmark: defensa contra malware](../security/benchmarks/security-control-malware-defense.md).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Microsoft Antimalware está habilitado en el host subyacente que admite los servicios de Azure (como Azure Synapse SQL), pero no se ejecuta en el contenido del cliente.

Examine previamente el contenido que se carga en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, Azure SQL Server, etc. Microsoft no tiene acceso a los datos de estas instancias.

- [Descripción de Microsoft Antimalware para Azure Cloud Services y Virtual Machines](../security/fundamentals/antimalware.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Azure Security Benchmark: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Las instantáneas del grupo de SQL dedicado se toman a lo largo del día y crean puntos de restauración que están disponibles durante siete días. No se puede cambiar este período de retención. El grupo de SQL dedicado admite un objetivo de punto de recuperación (RPO) de ocho horas. Puede restaurar el almacenamiento de datos de la región primaria a partir de cualquiera de las instantáneas capturadas en los últimos siete días. Tenga en cuenta que también puede desencadenar manualmente las instantáneas si es necesario.

- [Copia de seguridad y restauración en un grupo de SQL dedicado](sql-data-warehouse/backup-and-restore.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 9.1](../../includes/policy/standards/asb/rp-controls/microsoft.sql-9-1.md)]

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Realización de copias de seguridad completas del sistema y copias de seguridad de las claves administradas por el cliente

**Guía**: Las instantáneas del almacenamiento de datos se toman automáticamente a lo largo del día y crean puntos de restauración que están disponibles durante siete días. No se puede cambiar este período de retención. El grupo de SQL dedicado admite un objetivo de punto de recuperación (RPO) de ocho horas. Puede restaurar el almacenamiento de datos de la región primaria a partir de cualquiera de las instantáneas capturadas en los últimos siete días. Tenga en cuenta que también puede desencadenar manualmente las instantáneas si es necesario.

Si usa una clave administrada por el cliente para cifrar la clave de cifrado de base de datos, asegúrese de que se está realizando una copia de seguridad de la clave.

- [Copia de seguridad y restauración en un grupo de SQL dedicado](sql-data-warehouse/backup-and-restore.md)

- [Realización de copias de seguridad de claves de Azure Key Vault](/powershell/module/az.keyvault/backup-azkeyvaultkey)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Sql**:

[!INCLUDE [Resource Policy for Microsoft.Sql 9.2](../../includes/policy/standards/asb/rp-controls/microsoft.sql-9-2.md)]

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: Pruebe periódicamente los puntos de restauración para asegurarse de que las instantáneas son válidas. Para restaurar un grupo de SQL dedicado existente a partir de un punto de restauración, puede usar Azure Portal o PowerShell. Pruebe la restauración de la copia de seguridad de las claves administradas por el cliente.

- [Cómo restaurar claves de Azure Key Vault](/powershell/module/az.keyvault/restore-azkeyvaultkey)

- [Copia de seguridad y restauración en un grupo de SQL dedicado](sql-data-warehouse/backup-and-restore.md)

- [Cómo restaurar un grupo de SQL dedicado existente](sql-data-warehouse/sql-data-warehouse-restore-active-paused-dw.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: En Azure SQL Database, puede configurar una base de datos única o agrupada con una directiva de retención de copias de seguridad a largo plazo (LTR) para retener automáticamente las copias de seguridad de las bases de datos en contenedores de Azure Blob Storage hasta 10 años. Los datos de Azure Storage se cifran y descifran de forma transparente mediante el cifrado AES de 256 bits, uno de los cifrados de bloques más sólidos que hay disponibles, y son compatibles con FIPS 140-2.

De manera predeterminada, los datos de una cuenta de almacenamiento se cifran con claves administradas por Microsoft. Puede confiar en las claves administradas por Microsoft para el cifrado de los datos, o puede administrar el cifrado con sus propias claves. Si administra sus propias claves con Key Vault, asegúrese de que la eliminación temporal está habilitada.

- [Administración de la retención de copias de seguridad a largo plazo de Azure SQL Database](../azure-sql/database/long-term-backup-retention-configure.md)

- [Cifrado de Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md)

- [Habilitación de la eliminación temporal en Key Vault](../storage/blobs/soft-delete-blob-overview.md)

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

**Instrucciones**: Security Center asigna una gravedad a las alertas, que le ayudan a priorizar el orden en el que asiste a cada alerta, de modo que cuando un recurso está en peligro, puede obtenerlo inmediatamente. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en la métrica utilizados para emitir la alerta, así como en el nivel de confianza de que se produjo un intento malintencionado detrás de la actividad que provocó la alerta.

- [Alertas de seguridad en el Centro de seguridad de Azure](../security-center/security-center-alerts-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Guía**: Realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan según sea necesario.

- [Puede consultar la publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: la información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos.

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

**Guía**: siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft: https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.

- [Aquí encontrará más información sobre la estrategia y puesta en marcha de un equipo rojo de Microsoft, y las pruebas de penetración en sitios activos de la infraestructura en la nube, las aplicaciones y los servicios administrados por Microsoft](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e).

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [Información general sobre Azure Security Benchmark V2](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
