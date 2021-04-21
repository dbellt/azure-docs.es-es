---
title: Base de referencia de seguridad de Azure para Microsoft Azure Peering Service
description: La base de referencia de seguridad de Microsoft Azure Peering Service proporciona una guía de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: peering-service
ms.topic: conceptual
ms.date: 04/09/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 854c669861c6aff71861481843bff97fdb7157ab
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107315356"
---
# <a name="azure-security-baseline-for-microsoft-azure-peering-service"></a>Base de referencia de seguridad de Azure para Microsoft Azure Peering Service

Esta base de referencia de seguridad aplica la guía de la [versión 2.0 de Azure Security Benchmark](../security/benchmarks/overview.md) en Microsoft Azure Peering Service. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa por medio de los **controles de seguridad** que se definen en Azure Security Benchmark y las directrices relacionadas aplicables a Microsoft Azure Peering Service.

> [!NOTE]
> Se han excluido los **controles** que no son aplicables a Microsoft Azure Peering Service o cuya responsabilidad es de Microsoft. Para ver cómo Microsoft Azure Peering Service se asigna por completo a Azure Security Benchmark, consulte el **[archivo de asignación completo de base de referencia de seguridad de Microsoft Azure Peering Service](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines)** .

## <a name="network-security"></a>Seguridad de redes

*Para más información, consulte [Azure Security Benchmark: seguridad de red](/azure/security/benchmarks/security-controls-v2-network-security).*

### <a name="ns-5-deploy-intrusion-detectionintrusion-prevention-systems-idsips"></a>NS-5: implementación de sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS)

**Guía:** Peering Service no ofrece servicios nativos de detección o prevención de intrusiones. Los clientes deben seguir los procedimientos recomendados de seguridad, como el uso del filtrado basado en la inteligencia sobre amenazas de Azure Firewall para alertar y bloquear el tráfico desde y hacia direcciones IP y dominios malintencionados conocidos, según corresponda a sus requisitos empresariales. La direcciones IP y los dominios proceden de la fuente de Inteligencia sobre amenazas de Microsoft. 

En los casos en que deba inspeccionar la carga útil, implemente desde Azure Marketplace un sistema de prevención o detección de intrusiones de terceros (IDS/IPS), con capacidades de inspección de la carga útil.  
Como alternativa, puede usar una solución de respuesta y detección de puntos de conexión (EDR) basada en el host, junto con sistemas de prevención o detección de intrusiones basados en la red; asimismo puede usar esta solución en lugar de estos sistemas.  

Si tiene un requisito normativo para usar un sistema de detección o prevención de intrusiones, asegúrese de que siempre esté optimizado para consumir o proporcionar alertas de alta calidad. 

- [Implementación de Azure Firewall](../firewall/tutorial-firewall-deploy-portal.md)

- [Azure Marketplace incluye funcionalidades de IDS de terceros](https://azuremarketplace.microsoft.com/marketplace?search=IDS)

- [Funcionalidad de detección y respuesta de la protección contra amenazas avanzada de Microsoft Defender](/windows/security/threat-protection/microsoft-defender-atp/overview-endpoint-detection-response)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="identity-management"></a>Administración de identidades

*Para más información, consulte [Azure Security Benchmark: Administración de identidades](/azure/security/benchmarks/security-controls-v2-identity-management).*

### <a name="im-1-standardize-azure-active-directory-as-the-central-identity-and-authentication-system"></a>IM-1: Unificación en Azure Active Directory como sistema central de identidad y autenticación

**Guía**: Mantenga un inventario de las cuentas de usuario que tienen acceso administrativo al plano de control (como Azure Portal) de los recursos de Microsoft Azure Peering Service.

Puede usar el panel de Administración de identidad y acceso (IAM) de Azure Portal para la suscripción, a fin de configurar el control de acceso basado en roles de Azure (Azure RBAC). Los roles se aplican a usuarios, grupos, entidades de servicio e identidades administradas en Azure Active Directory (Azure AD).

- [Incorporación o eliminación de asignaciones de roles de Azure con Azure Portal](../role-based-access-control/role-assignments-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-2-manage-application-identities-securely-and-automatically"></a>IM-2: Administración de identidades de aplicaciones de forma segura y automática

**Guía:** las identidades administradas no se admiten en Peering Service. En el caso de los servicios que no admiten identidades administradas como Peering Service, use Azure Active Directory (Azure AD) para crear entidades de servicio con permisos restringidos en el nivel de recurso. 

- [Identidades administradas de Azure](../active-directory/managed-identities-azure-resources/overview.md)

- [Servicios que admiten identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

- [Entidad de servicio de Azure](/powershell/azure/create-azure-service-principal-azureps)

- [Creación de una entidad de servicio con certificado](../active-directory/develop/howto-authenticate-service-principal-powershell.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-3-use-azure-ad-single-sign-on-sso-for-application-access"></a>IM-3: Uso del inicio de sesión único de Azure AD para acceder a las aplicaciones

**Guía**: Microsoft Azure Peering Service usa Azure Active Directory (Azure AD) para proporcionar administración de identidades y acceso en los recursos de Azure. Esto incluye las identidades empresariales, como los empleados, así como las identidades externas, como asociados y proveedores. 

Use el inicio de sesión único para administrar y proteger el acceso a los datos y los recursos de su organización locales y en la nube. Conecte todos los usuarios, las aplicaciones y los dispositivos a Azure AD para obtener un acceso seguro y sin problemas, y para conseguir más visibilidad y control.

- [Descripción del inicio de sesión único de aplicaciones con Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-4-use-strong-authentication-controls-for-all-azure-active-directory-based-access"></a>IM-4: Uso de controles con autenticación multifactor sólida para todo el acceso basado en Azure Active Directory

**Guía**: habilite la autenticación multifactor de Azure Active Directory (Azure AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center.

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-5-monitor-and-alert-on-account-anomalies"></a>IM-5: Supervisión y alerta de anomalías de cuenta

**Guía**: use Privileged Identity Management (PIM) con Azure Active Directory (Azure AD) para la generación de registros y alertas cuando se produzca una actividad sospechosa o insegura en el entorno. Además, use las detecciones de riesgo de Azure AD para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

- [Cómo implementar Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Información sobre las detecciones de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-6-restrict-azure-resource-access-based-on-conditions"></a>IM-6: Restricción del acceso a recursos de Azure en función de las condiciones

**Guía**: use el acceso condicional con Azure Active Directory (Azure AD) para obtener un control de acceso más granular basado en condiciones definidas por el usuario, como requerir inicios de sesión de usuario desde intervalos IP determinados para usar la autenticación multifactor. También se puede usar una administración de sesión de autenticación granular mediante la directiva de acceso condicional de Azure AD para distintos casos de uso. 

- [Introducción al acceso condicional de Azure](../active-directory/conditional-access/overview.md)

- [Directivas de acceso condicional habituales](../active-directory/conditional-access/concept-conditional-access-policy-common.md)

- [Frecuencia de inicio de sesión de usuario](../active-directory/conditional-access/howto-conditional-access-session-lifetime.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="im-7-eliminate-unintended-credential-exposure"></a>IM-7: Elimine la exposición de credenciales no intencionada

**Guía**: implemente el escáner de credenciales en Azure DevOps para identificar las credenciales que se usen en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault.

En GitHub, puede usar la característica de escaneo de secretos nativos para identificar credenciales u otro tipo de secretos en el código.

- [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

- [Escaneo de secretos de GitHub](https://docs.github.com/github/administering-a-repository/about-secret-scanning)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="privileged-access"></a>Acceso con privilegios

*Para más información, consulte [Azure Security Benchmark: Acceso con privilegios](/azure/security/benchmarks/security-controls-v2-privileged-access).*

### <a name="pa-1-protect-and-limit-highly-privileged-users"></a>PA-1: Protección y limitación de usuarios con privilegios elevados

**Guía**: Limite el número de cuentas de usuario con privilegios elevados y proteja dichas cuentas en un nivel elevado. 

Los roles integrados más críticos de Azure Active Directory (Azure AD) son el Administrador global y el Administrador de roles con privilegios, ya que los usuarios asignados a estos dos roles pueden delegar roles de administrador. 

Con estos privilegios, los usuarios pueden leer y modificar todos los recursos de su entorno de Azure de forma directa o indirecta:

- Administrador global/administrador de empresa: los usuarios con este rol tienen acceso a todas las características administrativas de Azure AD, así como a los servicios que utilizan identidades de Azure AD.

- Administrador de roles con privilegios: los usuarios con este rol pueden administrar asignaciones de roles en Azure AD, y dentro de Azure AD Privileged Identity Management (PIM). Además, este rol permite administrar todos los aspectos de PIM y de las unidades administrativas.

Si usa roles personalizados con determinados permisos con privilegios asignados, es posible que tenga otros roles críticos que deban administrarse. Además, puede aplicar controles similares a la cuenta de administrador de recursos empresariales críticos.  

Puede habilitar el acceso con privilegios Just-in-Time (JIT) a los recursos de Azure y a Azure AD mediante Azure AD Privileged Identity Management (PIM). JIT concede permisos temporales para realizar tareas con privilegios solo cuando los usuarios lo necesitan. PIM también puede generar alertas de seguridad cuando hay actividades sospechosas o no seguras en la organización de Azure AD.

- [Permisos de rol administrativo en Azure AD](/azure/active-directory/users-groups-roles/directory-assign-admin-roles)

- [Uso de alertas de seguridad de Azure Privileged Identity Management](../active-directory/privileged-identity-management/pim-how-to-configure-security-alerts.md)

- [Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD](/azure/active-directory/users-groups-roles/directory-admin-roles-secure)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-2-restrict-administrative-access-to-business-critical-systems"></a>PA-2: Restricción del acceso administrativo a los sistemas críticos para la empresa

**Guía**: aísle el acceso a los sistemas críticos para la empresa mediante la restricción de las cuentas a las que se concede acceso con privilegios a las suscripciones y a los grupos de administración en los que se encuentran. 

Asimismo, se restringe el acceso a los sistemas de administración, identidad y seguridad que tienen acceso administrativo a los recursos críticos para la empresa, como los controladores del dominio de Active Directory, las herramientas de seguridad y las herramientas de administración del sistema con agentes instalados en sistemas críticos para la empresa. Los atacantes que ponen en peligro estos sistemas de administración y seguridad pueden convertirlos inmediatamente en un arma para poner en peligro los recursos críticos para la empresa. 

Para garantizar un control de acceso coherente, todos los tipos de control de acceso se deben alinear con la estrategia de segmentación de la empresa. 

Asegúrese de asignar cuentas con privilegios independientes que sean distintas de las cuentas de usuario estándar que se usan para las tareas de correo electrónico, exploración y productividad.

- [Modelo de referencia y componentes de Azure](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151)

- [Acceso al grupo de administración](https://docs.microsoft.com/azure/governance/management-groups/overview#management-group-access)

- [Administradores de la suscripción de Azure](../cost-management-billing/manage/add-change-subscription-administrator.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-3-review-and-reconcile-user-access-regularly"></a>PA-3: Revisión y conciliación de manera periódica del acceso de los usuarios

**Guía**: Azure Active Directory (AD) proporciona registros para ayudar a descubrir cuentas obsoletas. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. El acceso de los usuarios se puede revisar de forma periódica para asegurarse de que solo las personas adecuadas tengan acceso continuado.

- [Búsqueda de los informes de actividades de Azure AD](../active-directory/reports-monitoring/howto-find-activity-reports.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-4-set-up-emergency-access-in-azure-ad"></a>PA-4: Configuración del acceso de emergencia en Azure AD

**Guía:** configure una cuenta de emergencia para el acceso cuando las cuentas administrativas normales no estén disponibles, y así evitar que se bloquee accidentalmente y se quede fuera de la organización de Azure Active Directory (Azure AD). Las cuentas de acceso de emergencia pueden contener privilegios de nivel superior y no deben asignarse a individuos específicos. Limite las cuentas de acceso de emergencia a escenarios de emergencia o escenarios "excepcionales".

Igualmente, debe asegurarse de que las credenciales (como la contraseña, el certificado o la tarjeta inteligente) de las cuentas de acceso de emergencia estén protegidas y solo las conozcan aquellas personas que estén autorizadas para usarlas solo en caso de emergencia.

- [Administración de cuentas de acceso de emergencia en Azure AD](/azure/active-directory/users-groups-roles/directory-emergency-access)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-5-automate-entitlement-management"></a>PA-5: Automatización de la administración de derechos 

**Guía**: use las características de administración de derechos de Azure Active Directory (Azure AD) para automatizar los flujos de trabajo de solicitud de acceso, como las asignaciones, las revisiones y la expiración del acceso. También se admite la aprobación en dos o varias fases.

- [¿Qué son las revisiones de acceso de Azure Active Directory (Azure AD)?](../active-directory/governance/access-reviews-overview.md)

- [¿Qué es la administración de derechos de Azure Active Directory (Azure AD)?](../active-directory/governance/entitlement-management-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-6-use-privileged-access-workstations"></a>PA-6: Uso de estaciones de trabajo con privilegios de acceso

**Guía**: use una estación de trabajo de acceso con privilegios (PAW) con la autenticación multifactor de Azure Active Directory (Azure AD) que esté habilitada para iniciar sesión y permitirle configurar los recursos relacionados con Azure Sentinel.

- [Uso de estaciones de trabajo con privilegios de acceso](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Planeamiento de una implementación de autenticación multifactor de Azure AD](../active-directory/authentication/howto-mfa-getstarted.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pa-7-follow-just-enough-administration-least-privilege-principle"></a>PA-7: Seguimiento de solo una administración suficiente (principio de privilegios mínimos) 

**Guía**: el control de acceso basado en roles de Azure (Azure RBAC) le permite administrar el acceso a los recursos de Azure a través de las asignaciones de roles. Puede asignar estos roles a usuarios, grupos, entidades de servicio e identidades administradas. Hay roles integrados predefinidos para determinados recursos, y estos roles se pueden inventariar o consultar mediante herramientas, como la CLI de Azure, Azure PowerShell y Azure Portal.

Los privilegios que se asignen a los recursos a través de Azure RBAC siempre se deben limitar a los requisitos de los roles. Asimismo, los privilegios limitados complementan el enfoque Just-in-Time (JIT) de Azure Active Directory (Azure AD) Privileged Identity Management (PIM), y esos privilegios deben revisarse periódicamente.

Use los roles integrados para asignar los permisos y crear solo el rol personalizado cuando sea necesario.

- [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../role-based-access-control/overview.md)

- [Configuración de Azure RBAC](../role-based-access-control/role-assignments-portal.md)

- [Procedimiento para usar las revisiones de acceso e identidades de Azure AD](../active-directory/governance/access-reviews-overview.md)

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

- [Información general sobre los grupos de administración de Azure](../governance/management-groups/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="am-2-ensure-security-team-has-access-to-asset-inventory-and-metadata"></a>AM-2: Asegurarse de que el equipo de seguridad tiene acceso a los metadatos y al inventario de recursos

**Guía**: asegúrese de que los equipos de seguridad tienen acceso a un inventario de recursos que se actualice continuamente en Azure. Los equipos de seguridad suelen necesitar este inventario para evaluar la exposición potencial de su organización a los riesgos emergentes y como una entrada para mejoras de seguridad continuas. 

La característica de inventario de Azure Security Center y Azure Resource Graph pueden consultar y detectar todos los recursos de las suscripciones, incluidos los recursos de red, las aplicaciones y los servicios de Azure.  

Organice de forma lógica los recursos según la taxonomía de su organización mediante etiquetas y otros metadatos en Azure (nombre, descripción y categoría).  

- [Creación de consultas con Azure Resource Graph Explorer](../governance/resource-graph/first-query-portal.md)

- [Administración del inventario de recursos de Azure Security Center](../security-center/asset-inventory.md)

- [Para más información sobre el etiquetado de recursos, vea la guía de decisiones de nomenclatura y etiquetado de recursos](https://docs.microsoft.com/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json).

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="am-3-use-only-approved-azure-services"></a>AM-3: Uso exclusivo de servicios de Azure aprobados

**Guía:** Microsoft Azure Peering Service admite implementaciones de Azure Resource Manager y el cumplimiento de la configuración mediante Azure Policy en el espacio de nombres "Microsoft.Peering/peerings". Use Azure Policy para auditar y restringir qué servicios pueden aprovisionar los usuarios en su entorno. Use Azure Resource Graph para consultar y detectar recursos dentro de sus suscripciones. También puede usar Azure Monitor para crear reglas para desencadenar alertas cuando se detecta un servicio no aprobado.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#general)

- [Creación de consultas con Azure Resource Graph Explorer](../governance/resource-graph/first-query-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="am-5-limit-users-ability-to-interact-with-azure-resource-manager"></a>AM-5: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Guía**: Configure el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management.

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="logging-and-threat-detection"></a>registro y detección de amenazas

*Para más información, consulte [Azure Security Benchmark: registro y detección de amenazas](/azure/security/benchmarks/security-controls-v2-logging-threat-protection).*

### <a name="lt-1-enable-threat-detection-for-azure-resources"></a>LT-1: Habilitación de la detección de amenazas para recursos de Azure

**Guía**: asegúrese de que está supervisando distintos tipos de recursos de Azure para detectar posibles amenazas y anomalías. Céntrese en obtener alertas de alta calidad para reducir los falsos positivos que deben revisar los analistas. Las alertas se pueden crear a partir de datos de registro, agentes u otros datos.

Use la funcionalidad integrada de detección de amenazas de Azure Security Center, que se basa en la supervisión de los datos de telemetría de los servicios de Azure y en el análisis de los registros de servicio. Los datos se recopilan con el agente de Log Analytics, que lee distintas configuraciones relacionadas con la seguridad y diversos registros de eventos del sistema y copia los datos en el área de trabajo para analizarlos. 

Además, use Azure Sentinel para compilar reglas de análisis, que capturan las amenazas que cumplen criterios específicos en su entorno. Las reglas generan incidentes cuando los criterios coinciden, por lo que puede investigar cada incidente. Azure Sentinel también puede importar inteligencia contra amenazas de terceros para mejorar su capacidad de detección de amenazas. 

- [Protección contra amenazas en Azure Security Center](/azure/security-center/threat-protection)

- [Guía de referencia de alertas de seguridad de Azure Security Center](../security-center/alerts-reference.md)

- [Creación de reglas de análisis personalizadas para detectar amenazas](../sentinel/tutorial-detect-threats-custom.md)

- [Inteligencia sobre amenazas cibernéticas con Azure Sentinel](/azure/architecture/example-scenario/data/sentinel-threat-intelligence)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="lt-2-enable-threat-detection-for-azure-identity-and-access-management"></a>LT-2: Habilitación de la detección de amenazas para la administración de identidades y acceso de Azure

**Guía**: Azure Active Directory (Azure AD) proporciona registros de usuario que se pueden ver con los informes de Azure AD o integrarse con Azure Monitor. Estos registros también se pueden integrar con Azure Sentinel u otra solución de administración de eventos e información de seguridad (SIEM) o mediante herramientas de supervisión para casos de uso de supervisión y análisis más sofisticados:

- Inicio de sesión: el informe de inicio de sesión proporciona información sobre el uso de aplicaciones administradas y actividades de inicio de sesión del usuario.

- Registros de auditoría: proporcionan rastreabilidad mediante los registros de todos los cambios realizados por diversas características de Azure AD. Algunos ejemplos de registros de auditoría incluyen los cambios realizados en cualquier recurso de Azure AD, como agregar o quitar usuarios, aplicaciones, grupos, roles y directivas.

- Inicios de sesión de riesgo: un inicio de sesión de riesgo es un indicador de un intento de inicio de sesión que puede haber realizado alguien que no es el propietario legítimo de una cuenta de usuario.

- Usuarios marcados en riesgo: un usuario en riesgo es un indicador de una cuenta de usuario que puede haber estado en peligro.

Azure Security Center también puede alertar sobre determinadas actividades sospechosas, como un número excesivo de intentos de autenticación incorrectos y cuentas en desuso en la suscripción. Además de la supervisión básica de la protección de seguridad, el módulo de protección contra amenazas de Security Center también puede recopilar alertas de seguridad adicionales más detalladas de recursos de procesos de Azure concretos (como máquinas virtuales, contenedores, App Service), recursos de datos (como bases de datos SQL y almacenamiento) y niveles de servicio de Azure. Esta funcionalidad permite ver anomalías de las cuentas dentro de los recursos individuales.

- [Informes de actividad de auditoría en Azure AD](../active-directory/reports-monitoring/concept-audit-logs.md)

- [Habilitación de Azure Identity Protection](../active-directory/identity-protection/overview-identity-protection.md)

- [Protección contra amenazas en Azure Security Center](/azure/security-center/threat-protection)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="lt-3-enable-logging-for-azure-network-activities"></a>LT-3: Habilitación del registro para las actividades de red de Azure

**Guía:** configure la telemetría de conexión para proporcionar información sobre la conectividad entre la ubicación de conexión y la

red de Microsoft a través de Microsoft Azure Peering Service.

- [Configuración de la telemetría de conexión](measure-connection-telemetry.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="lt-4-enable-logging-for-azure-resources"></a>LT-4: Habilitación del registro para recursos de Azure

**Guía**: habilite el registro de Azure Activity y envíe los registros a un área de trabajo de Log Analytics, un centro de eventos de Azure o una cuenta de almacenamiento de Azure para archivarlos. 

Los registros de actividad proporcionan información sobre las operaciones llevadas a cabo en los recursos de Azure ExpressRoute en el nivel del plano de control. Con los datos del registro de actividades de Azure, puede responder a las preguntas "qué, quién y cuándo" de las operaciones de escritura (PUT, POST, DELETE) que se llevan a cabo en el nivel del plano de control para los recursos.

- [Registro de actividad de Azure](/azure/azure-monitor/platform/activity-log)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="lt-5-centralize-security-log-management-and-analysis"></a>LT-5: Centralizar la administración y el análisis de los registros de seguridad

**Guía**: Centralice el almacenamiento y el análisis de los registros para permitir su correlación. Asegúrese de que, para cada origen de registro, asigna un propietario de datos, una guía de acceso y una ubicación de almacenamiento; que determinar qué herramientas se usan para procesar y acceder a los datos, y los requisitos de retención de datos.

Asegúrese también de que integra los registros de actividad de Azure en el registro central. Recopile registros mediante Azure Monitor para agregar datos de seguridad generados por dispositivos de punto de conexión, recursos de red y otros sistemas de seguridad. En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

Además, habilite e incorpore los datos a Azure Sentinel o a un sistema SIEM de terceros.

Muchas organizaciones optan por usar Azure Sentinel para los datos de acceso frecuente y Azure Storage para los datos inactivos que se usan con menos frecuencia.

- [Recopilación de registros y métricas de plataforma con Azure Monitor](../azure-monitor/essentials/diagnostic-settings.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="lt-6-configure-log-storage-retention"></a>LT-6: Configuración de la retención del almacenamiento de registros

**Guía**: en Azure Monitor, establezca el período de retención de registros en las áreas de trabajo de Log Analytics vinculadas a los recursos de Azure en función de las normativas de cumplimiento de la organización.

- [Establecimiento de parámetros de retención de registros](/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period)

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

- [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="ir-3-detection-and-analysis--create-incidents-based-on-high-quality-alerts"></a>IR-3: Detección y análisis: creación de incidentes en función de alertas de alta calidad 

**Guía**: Asegúrese de que cuenta con un proceso para crear alertas de alta calidad y medir la calidad de las alertas. Esto le permite aprender de incidentes anteriores y clasificar las alertas para los analistas, de modo que no pierdan tiempo con falsos positivos. 

Las alertas de alta calidad se pueden crear en función de la experiencia de pasados incidentes, información validada procedente de la comunidad y herramientas diseñadas para generar y limpiar alertas mediante la fusión y correlación de diversos orígenes de la señal. 

Azure Security Center proporciona alertas de alta calidad en muchos recursos de Azure. Puede usar el conector de datos de ASC para enviar las alertas a Azure Sentinel. Azure Sentinel le permite crear reglas de alerta avanzadas con el fin de generar automáticamente incidentes para investigar. 

Exporte las alertas y recomendaciones de Azure Security Center mediante la característica de exportación para ayudar a identificar riesgos en los recursos de Azure. Esta exportación puede hacerse de forma manual o de modo continuo.

- [Configuración de la exportación](../security-center/continuous-export.md)

- [Transmisión de alertas a Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="ir-4-detection-and-analysis--investigate-an-incident"></a>IR-4: Detección y análisis: investigación de incidentes

**Guía**: asegúrese de que los analistas pueden consultar y usar diversos orígenes de datos a medida que investigan posibles incidentes, para conseguir una visión global de lo que ha sucedido. Se deben recopilar diversos registros para realizar un seguimiento de las actividades de un posible atacante en la cadena de eliminación para evitar puntos ciegos.  También debe asegurarse de que se capturan detalles y aprendizajes para otros analistas y para futuras referencias históricas.  

Los orígenes de datos para la investigación incluyen los orígenes de registro centralizados que ya se recopilan de los servicios en el ámbito y los sistemas en ejecución, pero también pueden incluir:

- Datos de red: use registros de flujo de grupos de seguridad de red, Azure Network Watcher y Azure Monitor para capturar registros de flujo de red y otra información de análisis. 

- Instantáneas de sistemas en ejecución: 

    - Use la funcionalidad de instantáneas de la máquina virtual de Azure para crear una instantánea del disco del sistema en ejecución. 

    - Use la funcionalidad de volcado de la memoria nativa del sistema operativo para crear una instantánea de la memoria del sistema en ejecución.

    - Use la característica de instantánea de los servicios de Azure o la propia funcionalidad del software para crear instantáneas de los sistemas en ejecución.

Azure Sentinel proporciona amplios análisis de datos en prácticamente cualquier origen de registro y un portal de administración de casos para administrar todo el ciclo de vida de los incidentes. La información de inteligencia durante una investigación puede asociarse a un incidente con fines de seguimiento e informes. 

- [Instantánea del disco de una máquina Windows](../virtual-machines/windows/snapshot-copy-managed-disk.md)

- [Instantánea del disco de una máquina Linux](../virtual-machines/linux/snapshot-copy-managed-disk.md)

- [Recopilación del volcado de memoria e información de diagnóstico del servicio de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) 

- [Investigación de incidentes con Azure Sentinel](../sentinel/tutorial-investigate-cases.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="ir-5-detection-and-analysis--prioritize-incidents"></a>IR-5: Detección y análisis: clasificar incidentes

**Guía**: Proporcione contexto a los analistas sobre los incidentes en los que deberán centrarse en primer lugar en función de la gravedad de la alerta y la confidencialidad de los recursos. 

Azure Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.

Adicionalmente, marque los recursos con etiquetas y cree un sistema de nomenclatura para identificar y clasificar los recursos de Azure, especialmente los que procesan datos confidenciales.  Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente.

- [Alertas de seguridad en el Centro de seguridad de Azure](../security-center/security-center-alerts-overview.md)

- [Uso de etiquetas para organizar los recursos de Azure](/azure/azure-resource-manager/resource-group-using-tags)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="ir-6-containment-eradication-and-recovery--automate-the-incident-handling"></a>IR-6: Contención, erradicación y recuperación: automatización del control de incidentes

**Guía**: Automatice las tareas repetitivas manuales para acelerar el tiempo de respuesta y reducir la carga de los analistas. Las tareas manuales tardan más tiempo en ejecutarse, lo que ralentiza cada incidente y reduce el número de incidentes que un analista puede manejar. Las tareas manuales también aumentan la fatiga del analista, lo que aumenta el riesgo del error humano que produce retrasos y reduce la capacidad de los analistas de centrarse de manera efectiva en tareas complejas. Use las características de automatización de flujo de trabajo de Azure Security Center y Azure Sentinel para desencadenar acciones automáticamente o ejecutar un cuaderno de estrategias para responder a las alertas de seguridad entrantes. El cuaderno de estrategias lleva a cabo acciones, como el envío de notificaciones, la deshabilitación de cuentas y el aislamiento de redes problemáticas. 

- [Configuración de la automatización de flujos de trabajo en Security Center](../security-center/workflow-automation.md)

- [Configuración de respuestas automatizadas frente a amenazas en Azure Security Center](https://docs.microsoft.com/azure/security-center/tutorial-security-incident#triage-security-alerts)

- [Configuración de respuestas automatizadas frente a amenazas en Azure Sentinel](../sentinel/tutorial-respond-threats-playbook.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="posture-and-vulnerability-management"></a>administración de posturas y vulnerabilidades

*Para más información, consulte [Azure Security Benchmark: administración de posturas y vulnerabilidades](/azure/security/benchmarks/security-controls-v2-vulnerability-management).*

### <a name="pv-1-establish-secure-configurations-for-azure-services"></a>PV-1: establecimiento de configuraciones seguras para servicios de Azure 

**Guía**: Defina los límites de protección de seguridad para los equipos de infraestructura y DevOps al facilitar la configuración segura de los servicios de Azure que usan. 

Inicie la configuración de seguridad de los servicios de Azure con las líneas de base de Azure Security Benchmark y personalícelas según sea necesario para su organización. 

Use Azure Security Center para configurar Azure Policy con el fin de auditar y aplicar las configuraciones de los recursos de Azure. 

Puede usar Azure Blueprints para automatizar la implementación y configuración de servicios y entornos de aplicaciones, incluidas las plantillas de Resource Manager, los controles de Azure RBAC y las directivas, en una única definición de plano técnico.

- [Ilustración de la implementación de límites de protección en la zona de aterrizaje de escala empresarial](/azure/cloud-adoption-framework/ready/enterprise-scale/architecture#landing-zone-expanded-definition)

- [Uso de directivas de seguridad en Azure Security Center](../security-center/tutorial-security-policy.md)

- [Creación y administración de directivas para aplicar el cumplimiento](../governance/policy/tutorials/create-and-manage.md)

- [Azure Blueprints](../governance/blueprints/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pv-2-sustain-secure-configurations-for-azure-services"></a>PV-2: sostenimiento de configuraciones seguras para servicios de Azure

**Guía**: no es aplicable; esta recomendación está pensada para los recursos de proceso.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="pv-8-conduct-regular-attack-simulation"></a>PV-8: realización de una simulaciones de ataques periódicas

**Guía**: Según sea necesario, realice pruebas de penetración o actividades de equipo rojo en los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos.
siga las reglas de compromiso de la prueba de penetración de Microsoft Cloud para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de las pruebas de penetración del equipo rojo y sitios activos en la infraestructura de nube, los servicios y las aplicaciones administradas por Microsoft.

- [Pruebas de penetración en Azure](../security/fundamentals/pen-testing.md)

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
- [Recomendación para la arquitectura de seguridad de Azure: almacenamiento, datos y cifrado](https://docs.microsoft.com/azure/architecture/framework/security/storage-data-encryption?toc=/security/compass/toc.json&amp;bc=/security/compass/breadcrumb/toc.json)

- [Aspectos básicos de la seguridad de Azure: seguridad, cifrado y almacenamiento de datos de Azure](../security/fundamentals/encryption-overview.md)

- [Cloud Adoption Framework: procedimientos recomendados de cifrado y seguridad de los datos de Azure](https://docs.microsoft.com/azure/security/fundamentals/data-encryption-best-practices?toc=/azure/cloud-adoption-framework/toc.json&amp;bc=/azure/cloud-adoption-framework/_bread/toc.json)

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

- [Azure Network Security Overview (Información general sobre Azure Network Security)](../security/fundamentals/network-overview.md)

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

- [Información general sobre seguridad de administración de identidades de Azure](../security/fundamentals/identity-management-overview.md)

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
