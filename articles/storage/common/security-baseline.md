---
title: Línea de base de seguridad de Azure Storage
description: La línea de base de seguridad de Azure Storage proporciona una guía de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: storage
ms.topic: conceptual
ms.date: 03/16/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 28d5bdc788e3b292545fd4c016fae36149f3a600
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107589455"
---
# <a name="azure-security-baseline-for-azure-storage"></a>Línea de base de seguridad de Azure Storage

Esta línea de base de seguridad aplica las instrucciones de la [versión 1.0 de Azure Security Benchmark](../../security/benchmarks/overview-v1.md) a Azure Storage. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure.
El contenido se agrupa por medio de los **controles de seguridad** definidos por Azure Security Benchmark y las directrices aplicables a Azure Storage. Se han excluido los **controles** no aplicables a Azure Storage.

 
Para ver cómo Azure Storage se asigna por completo a Azure Security Benchmark, consulte el [archivo completo de asignación de línea de base de seguridad de Azure Storage](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Seguridad de redes

*Para más información, consulte [Azure Security Benchmark: seguridad de red](../../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: Para configurar el firewall de la cuenta de almacenamiento, restrinja el acceso a los clientes de intervalos específicos de direcciones IP públicas, redes virtuales concretas de Azure, o bien a recursos específicos de Azure.  También puede configurar puntos de conexión privados para que el tráfico al servicio de almacenamiento de la empresa se mueva exclusivamente mediante redes privadas.

Nota: Las cuentas de almacenamiento clásicas no admiten firewalls y redes virtuales.

- [Configuración del firewall de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-network-security#change-the-default-network-access-rule)

- [Configuración de puntos de conexión privados para Azure Storage](storage-private-endpoints.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Storage**:

[!INCLUDE [Resource Policy for Microsoft.Storage 1.1](../../../includes/policy/standards/asb/rp-controls/microsoft.storage-1-1.md)]

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes e interfaces de red

**Guía**: Azure Storage proporciona un modelo de seguridad por capas. Puede limitar el acceso a su cuenta de almacenamiento a las solicitudes procedentes de direcciones IP especificadas, intervalos IP o una lista de subredes de instancias de una instancia de Azure Virtual Network. Puede usar Azure Security Center y seguir las recomendaciones de protección de redes para ayudar a proteger sus recursos de red en Azure. Además, habilite los registros de flujos de grupos de seguridad de red para las redes virtuales o la subred configuradas para las cuentas de Storage mediante el firewall de la cuenta de Storage, y envíe registros a una cuenta de Storage para realizar la auditoría de tráfico. 

 
Tenga en cuenta que, si tiene puntos de conexión privados conectados a la cuenta de almacenamiento, no puede configurar reglas del grupo de seguridad de red para las subredes. 
 

 
- [Configuración de redes virtuales y firewalls de Azure Storage](storage-network-security.md) 
 

 
- [Habilitación de los registros de flujo de NSG](../../network-watcher/network-watcher-nsg-flow-logging-portal.md) 
 

 
- [Descripción de la seguridad de red proporcionada por Azure Security Center](../../security-center/security-center-network-recommendations.md) 
 

 
- [Descripción de los puntos de conexión privados para Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-private-endpoints#known-issues)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: No aplicable; la recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Denegación de las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: Habilite Azure Defender para Storage en la cuenta de Azure Storage. Azure Defender para Storage proporciona una capa adicional de inteligencia de seguridad que detecta intentos poco habituales y potencialmente peligrosos de acceder a las cuentas de almacenamiento o vulnerarlas.  Las alertas integradas de Azure Security Center se basan en las actividades para las que la comunicación de red estaba asociada con una dirección IP que se resolvió correctamente, tanto si la dirección IP es una dirección de riesgo conocida (por ejemplo, un criptominero conocido) como si se trata de una dirección IP que anteriormente no se había reconocido como de riesgo. Las alertas de seguridad se desencadenan cuando se producen anomalías en una actividad. 

- [Configuración de Azure Defender para Storage](azure-defender-storage-configure.md) 

- [Descripción de la inteligencia sobre amenazas integrada de Azure Security Center](../../security-center/azure-defender.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Guía**: La captura de paquetes de Network Watcher permite crear sesiones de captura para realizar el seguimiento del tráfico entre la cuenta de Storage y una máquina virtual. La sesión de captura cuenta con filtros para asegurarse de capturar solo el tráfico que se desea. La captura de paquetes ayuda a diagnosticar anomalías de la red, tanto de forma activa como reactiva. Otros usos son la recopilación de estadísticas de red, la obtención de información sobre las intrusiones de red y la depuración de las comunicaciones cliente-servidor, entre otros. Esta funcionalidad permite desencadenar capturas de paquetes de forma remota, lo que reduce la carga de tener que ejecutar una captura de paquetes manualmente y en la máquina virtual deseada, y permite ahorrar tiempo. 

- [ Administración de capturas de paquetes con Azure Network Watcher mediante Azure Portal](../../network-watcher/network-watcher-packet-capture-manage-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implementación de sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Guía**: Azure Defender para Storage proporciona una capa adicional de inteligencia de seguridad que detecta intentos poco habituales y potencialmente peligrosos de acceder a las cuentas de almacenamiento o vulnerarlas. Las alertas de seguridad se desencadenan cuando se producen anomalías en una actividad. Estas alertas de seguridad se integran en Azure Security Center y también se envían por correo electrónico a los administradores de las suscripciones, con detalles de la actividad sospechosa y recomendaciones sobre cómo investigar y solucionar las amenazas. 

- [Configuración de Azure Defender para Storage](https://docs.microsoft.com/azure/storage/common/azure-defender-storage-configure?tabs=azure-security-center)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="17-manage-traffic-to-web-applications"></a>1.7: Administre el tráfico a las aplicaciones web

**Guía**: No aplicable; la recomendación está pensada para las aplicaciones web que se ejecutan en Azure App Service o en recursos de proceso.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: En el caso de los recursos en instancias de Virtual Network que necesitan acceder a la cuenta de Storage, use etiquetas de servicio de Virtual Network para la red virtual configurada con el fin de definir los controles de acceso de red en grupos de seguridad de red o Azure Firewall. Puede utilizar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (por ejemplo, Storage) en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian. 

Cuando el acceso a la red se deba restringir al ámbito de determinadas cuentas de almacenamiento, use directivas de punto de conexión de servicio de Virtual Network.

- [Más información sobre el uso de etiquetas de servicio](../../virtual-network/service-tags-overview.md)

- [Más información sobre directivas de punto de conexión de servicio de Virtual Network para Azure Storage](../../virtual-network/virtual-network-service-endpoint-policies-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: Defina e implemente configuraciones de seguridad estándar para los recursos de red asociados con la cuenta de Azure Storage con Azure Policy. Use alias de Azure Policy en los espacios de nombres "Microsoft.Storage" y "Microsoft.Network" para crear directivas personalizadas con el fin de auditar o aplicar la configuración de red de los recursos de la cuenta de Storage. 

También puede usar definiciones de directivas integradas relacionadas con la cuenta de almacenamiento, como: Las cuentas de almacenamiento deben usar un punto de conexión del servicio de red virtual 

- [Configuración y administración de Azure Policy](../../governance/policy/tutorials/create-and-manage.md) 

- [Ejemplos de Azure Policy para el almacenamiento](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#storage) 

- [Ejemplos de Azure Policy para las redes](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#network) 

- [Creación de un plano técnico de Azure](../../governance/blueprints/create-blueprint-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: use etiquetas para grupos de seguridad de red y otros recursos relacionados con la seguridad de red y el flujo de tráfico. Gracias al etiquetado, podrá asociar pares de nombre-valor personalizados e integrados con un recurso de red determinado, lo que lo ayudará a organizar los recursos de red y a relacionar los recursos de Azure con el diseño de la red.

Use cualquiera de las definiciones de Azure Policy integradas relacionadas con el etiquetado, como "Requerir etiqueta y su valor", para asegurarse de que todos los recursos se creen con etiquetas y para notificarle los recursos no etiquetados existentes. 

Los grupos de seguridad de red admiten etiquetas, al contrario que las reglas de seguridad individuales. Las reglas de seguridad tienen un campo **Descripción** que puede usar para almacenar parte de la información que pondría normalmente en una etiqueta.

- [Creación y uso de etiquetas de recurso](../../azure-resource-manager/management/tag-resources.md)

- [Filtrado del tráfico de red con grupos de seguridad de red](../../virtual-network/tutorial-filter-network-traffic.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Instrucciones**: Use Azure Policy para registrar los cambios de configuración de los recursos de red.  Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos.  

 
- [Configuración y administración de Azure Policy](../../governance/policy/tutorials/create-and-manage.md)
 
 
- [Creación de alertas en Azure Monitor](../../azure-monitor/alerts/alerts-activity-log.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Azure Security Benchmark: registro y supervisión](../../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Instrucciones**: Ingiera recursos mediante Azure Monitor para agregar datos de seguridad agregados generados por dispositivos de punto de conexión, recursos de red y otros sistemas de seguridad. En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y utilice cuentas de Azure Storage para el almacenamiento de archivos a largo plazo, lo que puede incluir, opcionalmente, características de seguridad como almacenamiento inmutable y aplicación de suspensiones de retención.

 
- [Recopilación de registros y métricas de plataforma con Azure Monitor](../../azure-monitor/essentials/diagnostic-settings.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: Azure Storage Analytics proporciona registros para blobs, colas y tablas. Puede usar Azure Portal para configurar los registros que se registran para su cuenta.   

 
- [Configuración de la supervisión para la cuenta de Azure Storage](manage-storage-analytics-logs.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: Al almacenar los registros de eventos de seguridad en la cuenta de Azure Storage o el área de trabajo de Log Analytics, puede establecer la directiva de retención según los requisitos de la organización.  

 
- [Configuración de la directiva de retención para los registros de la cuenta de Azure Storage](https://docs.microsoft.com/azure/storage/common/manage-storage-analytics-logs#configure-logging) 

 
- [Cambio del período de retención de datos en Log Analytics](https://docs.microsoft.com/azure/azure-monitor/logs/manage-cost-storage#change-the-data-retention-period)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y revisión de registros

**Guía**: Para revisar los registros de Azure Storage, están las opciones habituales, como las consultas mediante la oferta de Log Analytics, así como una opción única de ver los archivos de registro directamente. En Azure Storage, los registros se almacenan en blobs a los que se debe acceder directamente desde `http://accountname.blob.core.windows.net/$logs`. La carpeta de registro está oculta de forma predeterminada, por lo que tendrá que desplazarse hasta ella directamente. No se mostrará en los comandos de la lista. 

 
Además, habilite Azure Defender para Storage en la cuenta de almacenamiento. Azure Defender para Storage proporciona una capa adicional de inteligencia de seguridad que detecta intentos poco habituales y potencialmente peligrosos de acceder a las cuentas de almacenamiento o vulnerarlas. Las alertas de seguridad se desencadenan cuando se producen anomalías en una actividad. Estas alertas de seguridad se integran en Azure Security Center y también se envían por correo electrónico a los administradores de las suscripciones, con detalles de la actividad sospechosa y recomendaciones sobre cómo investigar y solucionar las amenazas. 
 

 
- [Registro y revisión de datos](https://docs.microsoft.com/azure/storage/common/storage-analytics-logging#how-logs-are-stored) 
 

 
- [Configuración de Azure Defender para Storage](https://docs.microsoft.com/azure/storage/common/azure-defender-storage-configure?tabs=azure-portal)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: En Azure Security Center, habilite Azure Defender en la cuenta de Storage. Habilite la configuración de diagnóstico para la cuenta de Storage y envíe registros a un área de trabajo de Log Analytics. Incorpore el área de trabajo de Log Analytics a Azure Sentinel, ya que proporciona una solución de respuesta automatizada de orquestación de seguridad (SOAR). Esto permite crear cuadernos de estrategias (soluciones automatizadas) y usarlos para corregir problemas de seguridad.  

 
- [Incorporación de Azure Sentinel](../../sentinel/quickstart-onboard.md)  

 
- [Administración de alertas de seguridad en Azure Security Center](../../security-center/security-center-managing-and-responding-alerts.md)  

 
- [Alertas sobre datos de registro de Log Analytics](../../azure-monitor/alerts/tutorial-response.md)  

 
- [Registro de Azure Storage Analytics](storage-analytics-logging.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: Use Azure Security Center y habilite Azure Defender para Storage con el fin de detectar cargas de malware en Azure Storage mediante el análisis de reputación de hash y accesos sospechosos desde un nodo de salida de Tor activo (un proxy de anonimización). 

- [Configuración de Azure Defender para Storage](azure-defender-storage-configure.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: La solución Azure DNS Analytics (versión preliminar) de Azure Monitor recopila conclusiones en la infraestructura de DNS sobre seguridad, rendimiento y operaciones.  Actualmente, no se admiten las cuentas de Azure Storage, pero puede usar una solución de registro de DNS de terceros.  

 
- [Recopilación de información sobre la infraestructura de DNS con la solución DNS Analytics en versión preliminar](../../azure-monitor/insights/dns-analytics.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para más información, consulte [Azure Security Benchmark: identidad y control de acceso](../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: Azure Active Directory (Azure AD) tiene roles integrados que se deben asignar explícitamente y son consultables. Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc para detectar cuentas que son miembros de grupos administrativos.

- [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: Ni las cuentas de Azure Storage ni Azure Active Directory (Azure AD) tienen el concepto de contraseñas en blanco o predeterminadas. Azure Storage implementa un modelo de control de acceso que admite el control de acceso basado en roles de Azure (Azure RBAC), así como la clave compartida y las firmas de acceso compartido (SAS). Una característica de la autenticación con clave compartida y SAS es que no se asocia ninguna identidad con el autor de la llamada y, en consecuencia, no se puede realizar la autorización basada en permisos de la entidad de seguridad.

- [Autorización del acceso a datos en Azure Storage](storage-auth.md)

- [Descripción de las entidades de seguridad y el control de acceso para la cuenta de Azure Storage](storage-introduction.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Cree procedimientos operativos estándar en torno al uso de cuentas administrativas dedicadas que tengan acceso a la cuenta de Storage. Use la administración de identidad y acceso de Azure Security Center para supervisar el número de cuentas administrativas.

También puede habilitar el acceso Just-in-Time/Just-Enough usando roles con privilegios de Privileged Identity Management de Azure Active Directory (Azure AD) para los servicios de Microsoft y Azure Resource Manager.

- [Descripción de identidad y acceso en Azure Security Center](../../security-center/security-center-identity-access.md)

- [Introducción a Privileged Identity Management](/azure/active-directory/privileged-identity-management/)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4: Uso del inicio de sesión único (SSO) de Azure Active Directory

**Guía**: Siempre que sea posible, use el inicio de sesión único de Azure Active Directory (Azure AD) en lugar de configurar credenciales independientes individuales por servicio. Use las recomendaciones de administración de identidades y acceso de Azure Security Center.
 

 
- [Descripción de SSO con Azure AD](../../active-directory/manage-apps/what-is-single-sign-on.md)
 

 
- [Autorización del acceso a datos en Azure Storage](storage-auth.md)
 

 
- [Autorización del acceso a blobs y colas con Azure AD](storage-auth-aad.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Uso de la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Habilite la autenticación multifactor de Azure Active Directory (Azure AD) y siga las recomendaciones de administración de identidades y acceso de Azure Security Center para ayudar a proteger los recursos de la cuenta de Storage.

- [Habilitación de la autenticación multifactor en Azure](../../active-directory/authentication/howto-mfa-getstarted.md)

- [Supervisión de la identidad y el acceso en Azure Security Center](../../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6: Uso de estaciones de trabajo seguras y administradas por Azure para realizar tareas administrativas

**Guía**: Utilice estaciones de trabajo de acceso con privilegios (PAW) que tengan configurada la autenticación multifactor para iniciar sesión en la cuenta de Storage y configurarla.  

 
- [Más información sobre las estaciones de trabajo con privilegios de acceso](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/) 

 
- [Habilitación de la autenticación multifactor en Azure](../../active-directory/authentication/howto-mfa-getstarted.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: Envíe alertas de detección de riesgo de Azure Security Center a Azure Monitor y configure alertas o notificaciones personalizadas mediante grupos de acciones. Habilite Azure Defender para Storage en la cuenta de Storage con el fin de generar alertas de actividades sospechosas. Use la característica de detecciones de riesgo de Azure Active Directory (Azure AD) para ver alertas e informes sobre el comportamiento de los usuarios de riesgo.

- [Configuración de Azure Defender para Storage](azure-defender-storage-configure.md)
 

- [Información sobre las detecciones de riesgo de Azure AD](../../active-directory/identity-protection/overview-identity-protection.md)
 

- [Configuración de grupos de acciones para alertas y notificaciones personalizadas](../../azure-monitor/alerts/action-groups.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8: Administre los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: Use ubicaciones con nombre de acceso condicional para permitir el acceso solo desde agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones. 

- [Configuración de ubicaciones con nombre en Azure](../../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Instrucciones**: Use Azure Active Directory (AD) como sistema central de autenticación y autorización. Azure proporciona control de acceso basado en rol (Azure RBAC) para el control específico de acceso de los clientes a los recursos de una cuenta de almacenamiento.   Use credenciales de Azure AD cuando sea posible como procedimiento recomendado de seguridad, en lugar de usar la clave de cuenta, que se puede poner en peligro más fácilmente. Cuando el diseño de la aplicación requiera firmas de acceso compartido para el acceso a Blob Storage, use credenciales de Azure AD para crear firmas de acceso compartido (SAS) de delegación de usuarios cuando sea posible para una seguridad superior.

- [Procedimiento para crear y configurar una instancia de Azure AD](../../active-directory/fundamentals/active-directory-access-create-new-tenant.md) 

- [Uso del proveedor de recursos de Azure Storage para acceder a los recursos de administración](authorization-resource-provider.md)

- [Configuración del acceso a datos de colas y blobs de Azure con Azure RBAC en Azure Portal](storage-auth-aad-rbac-portal.md)

- [Autorización del acceso a datos en Azure Storage](storage-auth.md)

- [Grant limited access to Azure Storage resources using shared access signatures (SAS)](storage-sas-overview.md) (Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido [SAS])

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: Revise los registros de Azure Active Directory (Azure AD) con el fin de detectar cuentas obsoletas, como aquellas con roles administrativos de la cuenta de Storage. Además, use las revisiones de acceso de identidad de Azure para administrar de forma eficaz la pertenencia a grupos, el acceso a las aplicaciones empresariales que se pueden usar para acceder a recursos de cuentas de Storage y asignaciones de roles. El acceso de los usuarios se debe revisar de forma periódica para asegurarse de que solo los usuarios adecuados tengan acceso continuado. 
 

 
También puede usar una firma de acceso compartido (SAS) para ofrecer acceso delegado a los recursos en la cuenta de almacenamiento sin poner en peligro la seguridad de los datos. Puede controlar a qué recursos puede tener acceso el cliente, qué permisos tienen en esos recursos y cuánto tiempo es válida la SAS, entre otros parámetros.
 

 
Además, revise el acceso de lectura anónimo a contenedores y blobs. De forma predeterminada, solo un usuario al que se han concedido los permisos adecuados puede acceder a un contenedor y a los blobs dentro de él. Puede usar Azure Monitor para alertar sobre el acceso anónimo a cuentas de Storage mediante la condición de autenticación anónima.
 

 
Una manera eficaz de reducir el riesgo de acceso a cuentas de usuario no sospechosas es limitar la duración del acceso que se concede a los usuarios. Los URI de SAS de tiempo limitado son una manera eficaz de que caduque el acceso del usuario a una cuenta de Storage. Además, la rotación de claves de cuentas de Storage con frecuencia es una manera de asegurarse de que el acceso inesperado mediante las claves de la cuenta de Storage tiene una duración limitada.
 

 
- [Descripción de los informes de Azure AD](/azure/active-directory/reports-monitoring/) 
 

 
- [Visualización y modificación del acceso en el nivel de cuenta de Azure Storage](storage-auth-aad-rbac-portal.md)
 

 
- [Grant limited access to Azure Storage resources using shared access signatures (SAS)](storage-sas-overview.md) (Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido [SAS])
 

 
- [Administración del acceso de lectura anónimo a contenedores y blobs](../blobs/anonymous-read-access-configure.md)
 

 
- [Supervisión de una cuenta de almacenamiento en Azure Portal](manage-storage-analytics-logs.md)
 

 
- [Administración de las claves de acceso de la cuenta de almacenamiento](storage-account-keys-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: Use Storage Analytics para registrar información detallada sobre las solicitudes correctas y erróneas realizadas a un servicio de almacenamiento. Todos los registros se almacenan en blobs en bloques en un contenedor denominado $logs, que se crea automáticamente cuando se habilita Storage Analytics para una cuenta de almacenamiento.
 

Cree una configuración de diagnóstico para las cuentas de usuario de Azure Active Directory (Azure AD) mediante el envío de los registros de auditoría y los registros de inicio de sesión a un área de trabajo de Log Analytics. Puede configurar las alertas deseadas en el área de trabajo de Log Analytics.

Para supervisar los errores de autenticación en cuentas de Azure Storage, puede crear alertas que le avisen cuando se hayan alcanzado determinados umbrales para las métricas de recursos de almacenamiento. Además, use Azure Monitor para alertar sobre el acceso anónimo a cuentas de Storage mediante la condición de autenticación anónima.

- [Registro de Azure Storage Analytics](storage-analytics-logging.md)
 

- [Integración de los registros de actividad de Azure en Azure Monitor](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)
 

- [Configuración de alertas de métricas para cuentas de Azure Storage](manage-storage-analytics-logs.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12: Alerta de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Use las características de protección de identidad y detección de riesgo de Azure Active Directory (Azure AD) para configurar respuestas automatizadas a las acciones sospechosas que se detecten en relación con los recursos de las cuentas de Storage. Debe habilitar las respuestas automatizadas a través de Azure Sentinel para implementar las respuestas de seguridad de su organización.

- [Visualización de los inicios de sesión de riesgo de Azure AD](../../active-directory/identity-protection/overview-identity-protection.md)

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Incorporación de Azure Sentinel](../../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13: Proporcione a Microsoft acceso a los datos pertinentes del cliente durante los escenarios de soporte técnico

**Instrucciones**: En escenarios de soporte técnico en los que Microsoft necesita acceder a los datos del cliente, la Caja de seguridad del cliente (versión preliminar para cuentas de Storage) proporciona una interfaz para que los clientes revisen y aprueben o rechacen las solicitudes de acceso a los datos del cliente. Microsoft no necesitará ni solicitará acceso a los secretos de la organización almacenados en la cuenta de Storage.

- [Descripción de la Caja de seguridad del cliente](../../security/fundamentals/customer-lockbox-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-protection"></a>Protección de datos

*Para más información, consulte [Azure Security Benchmark: protección de datos](../../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Instrucciones**: Use etiquetas para ayudar a realizar el seguimiento de los recursos de cuentas de Storage que almacenan o procesan información confidencial. 

- [Creación y uso de etiquetas](../../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Guía**: Implemente el aislamiento mediante suscripciones independientes, grupos de administración y cuentas de Storage para dominios de seguridad individuales, como el entorno y la confidencialidad de los datos.  Puede restringir la cuenta de Storage para controlar el nivel de acceso a las cuentas de almacenamiento que exigen sus aplicaciones y entornos empresariales, en función del tipo y el subconjunto de redes usadas. Cuando se configuran las reglas de red, solo las aplicaciones que solicitan datos del conjunto especificado de redes pueden acceder a una cuenta de almacenamiento. Puede controlar el acceso a Azure Storage mediante Azure RBAC.

También puede configurar puntos de conexión privados para mejorar la seguridad cuando el tráfico entre la red virtual y el servicio atraviesa la red troncal de Microsoft, eliminando la exposición a la red pública de Internet.

- [Creación de suscripciones adicionales de Azure](../../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../../governance/management-groups/create-management-group-portal.md)

- [Creación y uso de etiquetas](../../azure-resource-manager/management/tag-resources.md)

- [Configuración de redes virtuales y firewalls de Azure Storage](storage-network-security.md)

- [Puntos de conexión de servicio de red virtual](../../virtual-network/virtual-network-service-endpoints-overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: En el caso de recursos de cuentas de Storage que almacenan o procesan información confidencial, marque los recursos como confidenciales mediante etiquetas. Para reducir el riesgo de pérdida de datos mediante filtración, restrinja el tráfico de red saliente para las cuentas de Azure Storage mediante Azure Firewall.  

Además, use directivas de punto de conexión de servicio de red virtual para filtrar el tráfico de red virtual de salida a cuentas de Azure Storage mediante un punto de conexión de servicio y habilite la filtración de datos únicamente para cuentas específicas de Azure Storage.

- [Configuración de redes virtuales y firewalls de Azure Storage](../../virtual-network/virtual-network-service-endpoint-policies-overview.md)

- [Directivas de punto de conexión de servicio de red virtual para Azure Storage](../../private-link/tutorial-private-endpoint-storage-portal.md)

- [Descripción de la protección de datos de los clientes en Azure](../../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Instrucciones**: Puede exigir el uso de HTTPS mediante la habilitación de la transferencia segura requerida para la cuenta de Storage. Una vez que esta opción esté habilitada, se rechazarán las conexiones que usan HTTP.  Además, use Azure Security Center y Azure Policy para aplicar la transferencia segura para la cuenta de Storage.

- [Requisito de transferencia segura en Azure Storage](storage-require-secure-transfer.md)

- [Directivas de seguridad de Azure supervisadas por Security Center](../../security-center/policy-reference.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.Storage**:

[!INCLUDE [Resource Policy for Microsoft.Storage 4.4](../../../includes/policy/standards/asb/rp-controls/microsoft.storage-4-4.md)]

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Guía**: Las características de identificación de datos todavía no están disponibles para la cuenta de Azure Storage y los recursos relacionados. Implemente una solución de terceros, si es necesario, para fines de cumplimiento. 

- [Descripción de la protección de datos de los clientes en Azure](../../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6: Uso del control de acceso basado en rol para controlar el acceso a los recursos

**Guía**: Azure Active Directory (Azure AD) autoriza derechos de acceso a recursos protegidos mediante el control de acceso basado en rol de Azure (Azure RBAC). Azure Storage define un conjunto de roles RBAC integrados de Azure que engloban los conjuntos comunes de permisos que se usan para acceder a los datos de los blobs o de las colas.

- [Asignación de roles RBAC de Azure a una cuenta de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac-portal#assign-azure-roles-using-the-azure-portal)

- [Uso del proveedor de recursos de Azure Storage para acceder a los recursos de administración](authorization-resource-provider.md)

- [Configuración del acceso a datos de colas y blobs de Azure con Azure RBAC en Azure Portal](storage-auth-aad-rbac-portal.md)

- [Procedimiento para crear y configurar una instancia de Azure AD](../../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

- [Autorización del acceso a datos en Azure Storage](storage-auth.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: El cifrado de Azure Storage está habilitado en todas las cuentas de almacenamiento y no se puede deshabilitar. Azure Storage cifra automáticamente los datos al guardarlos en la nube. Al leer datos de Azure Storage, Azure Storage los descifra antes de devolverlos. El cifrado de Azure Storage le permite proteger los datos en reposo sin tener que modificar el código o agregar código a las aplicaciones. 

- [Descripción del cifrado en reposo de Azure Storage](storage-service-encryption.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor con el registro de actividad de Azure para crear alertas para cuando se produzcan cambios en los recursos de la cuenta de Storage.  También puede habilitar el registro de Azure Storage para realizar un seguimiento de cómo se autorizó cada solicitud realizada en Azure Storage. Los registros indican si una solicitud se realizó de forma anónima o mediante un token OAuth 2.0, una clave compartida o una firma de acceso compartido (SAS).  Además, use Azure Monitor para alertar sobre el acceso anónimo a cuentas de Storage mediante la condición de autenticación anónima. 

 
- [Creación de alertas para los eventos del registro de actividad de Azure](../../azure-monitor/alerts/alerts-activity-log.md) 

 
- [Registro de Azure Storage Analytics](storage-analytics-logging.md) 

 
- [Configuración de alertas de métricas para cuentas de Azure Storage](manage-storage-analytics-logs.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Azure Security Benchmark: administración de vulnerabilidades](../../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Instrucciones**: Siga las recomendaciones de Azure Security Center para auditar y supervisar continuamente la configuración de las cuentas de almacenamiento.  

- [Guía de referencia sobre las recomendaciones de seguridad](../../security-center/recommendations-reference.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Guía**: No aplicable; Microsoft realiza la administración de vulnerabilidades en los sistemas subyacentes que admiten cuentas de Storage.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Instrucciones**: Use la clasificación de riesgo predeterminada (puntuación de seguridad) proporcionada por Azure Security Center. 

- [Descripción de la puntuación de seguridad de Azure Security Center](/azure/security-center/security-center-secure-score)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para más información, consulte [Azure Security Benchmark: inventario y administración de recursos](../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: Use Azure Resource Graph para consultar y detectar todos los recursos (incluidas las cuentas de Storage) de las suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

- [Creación de consultas con Azure Graph](../../governance/resource-graph/first-query-portal.md)

- [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription)

- [Descripción de Azure RBAC](../../role-based-access-control/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: Aplique etiquetas a los recursos de la cuenta de Storage que proporcionan metadatos para organizarlos de forma lógica en una taxonomía. 

- [Creación y uso de etiquetas](../../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Guía**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de las cuentas de Storage y los recursos relacionados. Concilie el inventario periódicamente y asegúrese de que los recursos no autorizados se eliminan de la suscripción de manera oportuna. 

Además, use Azure Defender para Storage con el fin de detectar recursos de Azure no autorizados. 

- [Creación de suscripciones adicionales de Azure](../../cost-management-billing/manage/create-subscription.md) 

- [Creación de grupos de administración](../../governance/management-groups/create-management-group-portal.md)

- [Creación y uso de etiquetas](../../azure-resource-manager/management/tag-resources.md)

- [Configuración de Azure Defender para Storage](azure-defender-storage-configure.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: Tendrá que crear un inventario de los recursos de Azure aprobados y el software aprobado para los recursos de proceso según las necesidades de la organización.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas: 

 
 - Tipos de recursos no permitidos 
 
 - Tipos de recursos permitidos 
 

 
Además, use Azure Resource Graph para consultar y detectar recursos en las suscripciones. Esto puede ayudar en entornos de alta seguridad, como aquellos con cuentas de Storage. 
 

 
- [Configuración y administración de Azure Policy](../../governance/policy/tutorials/create-and-manage.md) 
 

 
- [Creación de consultas con Azure Graph](../../governance/resource-graph/first-query-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: El cliente puede impedir la creación o el uso de recursos con Azure Policy según las directivas de la empresa del cliente. 

- [Configuración y administración de Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos
- Tipos de recursos permitidos

Información adicional disponible en los vínculos indicados.

- [Configuración y administración de Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#general)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy: Microsoft.ClassicStorage**:

[!INCLUDE [Resource Policy for Microsoft.ClassicStorage 6.9](../../../includes/policy/standards/asb/rp-controls/microsoft.classicstorage-6-9.md)]

**Definiciones integradas de Azure Policy: Microsoft.Storage**:

[!INCLUDE [Resource Policy for Microsoft.Storage 6.9](../../../includes/policy/standards/asb/rp-controls/microsoft.storage-6-9.md)]

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: Use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager configurando "Bloquear acceso" en la aplicación Microsoft Azure Management. Esto puede impedir la creación y los cambios en los recursos de un entorno de alta seguridad, como los que tienen cuentas de Storage. 

- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../../role-based-access-control/conditional-access-azure-management.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Azure Security Benchmark: configuración segura](../../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Use alias de Azure Policy en el espacio de nombres **Microsoft.Storage** para crear directivas personalizadas con el fin de auditar o aplicar la configuración de las instancias de la cuenta de Storage. También puede usar definiciones de Azure Policy para la cuenta de Azure Storage, como las siguientes:

- Auditar el acceso de red sin restricciones a cuentas de almacenamiento
- Implementación de Azure Defender para Storage
- Se deben migrar las cuentas de almacenamiento a los nuevos recursos de Azure Resource Manager
- Se debe habilitar la transferencia segura a las cuentas de almacenamiento

Use las recomendaciones de Azure Security Center como línea de base de configuración segura para las cuentas de Storage.

- [Visualización de los alias de Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

- [Configuración y administración de Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Use la instancia de Azure Policy [denegar] e [implementar si no existe] para aplicar la configuración segura en los recursos de la cuenta de Storage. 

- [Configuración y administración de Azure Policy](../../governance/policy/tutorials/create-and-manage.md) 

- [Descripción de los efectos de Azure Policy](../../governance/policy/concepts/effects.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: use Azure Repos para almacenar y administrar de forma segura el código, como directivas de Azure personalizadas, plantillas de Azure Resource Manager, scripts de Desired State Configuration, etc. Para acceder a los recursos que administra en Azure DevOps, puede conceder o denegar permisos a usuarios específicos, grupos de seguridad integrados o grupos definidos en Azure Active Directory (Azure AD) si se integran con Azure DevOps, o Azure AD si se integran con TFS.

- [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [Acerca de los permisos y los grupos en Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: Use Azure Policy para alertar, auditar y aplicar las configuraciones del sistema para la cuenta de Storage. Además, desarrolle un proceso y una canalización para administrar las excepciones de las directivas. 

- [ Configuración y administración de Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: Use Azure Security Center para realizar exámenes de base de referencia de sus recursos de la cuenta de Azure Storage. 

- [Corrección de recomendaciones en Azure Security Center](../../security-center/security-center-remediate-recommendations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: Azure Storage cifra automáticamente los datos al guardarlos en la nube. Puede usar las claves administradas por Microsoft para el cifrado de la cuenta de Storage o puede administrar el cifrado con sus propias claves. Si usa claves proporcionadas por el cliente, puede aprovechar Azure Key Vault para almacenarlas de forma segura. 

Además, cambie las claves de la cuenta de Storage con frecuencia para limitar el impacto de la pérdida o revelación de claves de la cuenta de Storage.

- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md)

- [Administración de las claves de acceso de la cuenta de almacenamiento](storage-account-keys-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: Autorice el acceso a blobs y colas desde las cuentas de Azure Storage con Azure Active Directory (Azure AD) e identidades administradas. Azure Blob Storage y Queue Storage admite la autenticación de Azure AD y con identidades administradas para recursos de Azure. 

Las identidades administradas para recursos de Azure pueden autorizar el acceso a datos de blobs y colas con credenciales de Azure AD desde aplicaciones que se ejecutan en máquinas virtuales de Azure, aplicaciones de función, conjuntos de escalado de máquinas virtuales y otros servicios. Si usa identidades administradas para recursos de Azure junto con autenticación de Azure AD, puede evitar el almacenamiento de credenciales con las aplicaciones que se ejecutan en la nube.

- [Concesión de acceso a los datos de blobs y colas de Azure mediante identidades administradas](storage-auth-aad-rbac-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault. 

- [ Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Azure Security Benchmark: defensa contra malware](../../security/benchmarks/security-control-malware-defense.md).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Use Azure Defender para Storage con el fin de detectar cargas de malware en Azure Storage mediante el análisis de reputación de hash y accesos sospechosos desde un nodo de salida de Tor activo (un proxy de anonimización). 
 

 
También puede realizar un examen previo de cualquier contenido en busca de malware antes de cargarlo en recursos de Azure que no son de proceso, como App Service, Data Lake Storage, Blob Storage, etc., para cumplir los requisitos de la organización.
 

 
- [Configuración de Azure Defender para Storage](azure-defender-storage-configure.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Azure Security Benchmark: recuperación de datos](../../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Los datos de la cuenta de Microsoft Azure Storage siempre se replican automáticamente para garantizar la durabilidad y la alta disponibilidad. Azure Storage copia los datos para protegerlos de eventos previstos e imprevistos, como errores transitorios del hardware, interrupciones del suministro eléctrico o de la red y desastres naturales masivos. Puede optar por replicar los datos en el mismo centro de datos, en centros de datos zonales que estén en la misma región o en regiones geográficamente separadas. 

También puede permitir que Azure Automation tome instantáneas periódicas de los blobs.

- [Descripción de la redundancia de Azure Storage y los acuerdos de nivel de servicio](storage-redundancy.md)

- [Crear una instantánea de un blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob)

- [Información general sobre Azure Automation](../../automation/automation-intro.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Para realizar una copia de seguridad de los datos de los servicios admitidos por la cuenta de Storage, hay varios métodos, como el uso de azcopy o herramientas de terceros. El almacenamiento inmutable para Azure Blob Storage permite a los usuarios almacenar objetos de datos críticos para la empresa en un estado WORM (escribir una vez, leer muchas). En este estado, los usuarios no pueden borrar ni modificar los datos durante el intervalo de tiempo especificado por el usuario.
 

Se pueden realizar copias de seguridad de las claves administradas o proporcionadas por el cliente en Azure Key Vault mediante la CLI de Azure o PowerShell. 

 
- [Introducción a AzCopy](storage-use-azcopy-v10.md)  

- [Establecimiento y administración de directivas de inmutabilidad para el almacenamiento de blobs](../blobs/storage-blob-immutability-policies-manage.md)
 

 
- [Creación de una copia de seguridad de las claves del almacén de claves en Azure](/powershell/module/az.keyvault/backup-azkeyvaultkey)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Instrucciones**: Realice restauraciones de datos automatizadas periódicas de sus certificados, claves, cuentas de almacenamiento administradas y secretos de Key Vault con los siguientes comandos de PowerShell:

Restore-AzKeyVaultCertificate

Restore-AzKeyVaultKey

Restore-AzKeyVaultManagedStorageAccount

Restore-AzKeyVaultSecret

- [Restauración de certificados de Key Vault](/powershell/module/az.keyvault/restore-azkeyvaultcertificate)

- [Restauración de claves de Key Vault](/powershell/module/az.keyvault/restore-azkeyvaultkey)

- [Restauración de cuentas de Storage de Key Vault](/powershell/module/az.keyvault/backup-azkeyvaultmanagedstorageaccount)

- [Restauración de secretos de Key Vault](/powershell/module/az.keyvault/restore-azkeyvaultsecret)

- [AzCopy es una utilidad de línea de comandos que puede usar para copiar blobs, archivos y datos de tabla a una cuenta de Storage o desde esta](storage-use-azcopy-v10.md)

Nota: Si desea copiar datos desde y hacia su servicio Azure Table Storage, instale AzCopy versión 7.3.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: Para habilitar las claves administradas por el cliente en una cuenta de almacenamiento, debe usar un almacén de Azure Key Vault para almacenar las claves. Debe habilitar las propiedades Eliminación temporal y No purgar en el almacén de claves.  La característica de eliminación temporal de Key Vault permite la recuperación de almacenes y objetos de almacén eliminados, como claves, secretos y certificados.  Si realiza una copia de seguridad de los datos de la cuenta de Storage en los blobs de Azure Storage, habilite la eliminación temporal para guardar y recuperar los datos cuando se eliminen blobs o instantáneas de blobs.  Debe tratar las copias de seguridad como datos confidenciales y aplicar los controles de protección de datos y de acceso pertinentes como parte de esta base de referencia.  Además, para mejorar la protección, puede almacenar objetos de datos críticos para la empresa en un estado WORMM (escribir una vez, leer muchas).

- [Uso de la eliminación temporal de Azure Key Vault](../../key-vault/general/key-vault-recovery.md)

- [Eliminación temporal de blobs de Azure Storage](../blobs/soft-delete-blob-overview.md)

- [Almacenamiento inmutable de los datos críticos para la empresa en Azure Blob Storage](../blobs/storage-blob-immutable-storage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Azure Security Benchmark: respuesta ante incidentes](../../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Guía del NIST para controlar incidentes de seguridad en equipos](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Instrucciones**: Security Center asigna una gravedad a cada alerta para ayudarle a priorizar aquellas que se deben investigar en primer lugar. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta. 

 
Además, marque claramente las suscripciones (por ejemplo, producción, no producción) con etiquetas y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure, especialmente los que procesan datos confidenciales.  Es su responsabilidad asignar prioridades a la corrección de las alertas en función de la importancia de los recursos y el entorno de Azure donde se produjo el incidente.
 

 
- [Alertas de seguridad en el Centro de seguridad de Azure](../../security-center/security-center-alerts-overview.md)
 

 
- [Uso de etiquetas para organizar los recursos de Azure](../../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Guía**: Realice ejercicios para probar las funcionalidades de respuesta a los incidentes de los sistemas periódicamente para ayudar a proteger los recursos de Azure. Identifique puntos débiles y brechas y revise el plan según sea necesario.

- [Publicación de NIST: Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Instrucciones**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos. Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

- [Establecimiento del contacto de seguridad de Azure Security Center](../../security-center/security-center-provide-security-contact-details.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Guía**: Exporte las alertas y recomendaciones de Azure Security Center y mediante la característica de exportación continua para ayudar a identificar los riesgos para los recursos de Azure. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. Puede usar el conector de datos de Azure Security Center para transmitir las alertas a Azure Sentinel.

- [Configuración de la exportación continua](../../security-center/continuous-export.md)

- [Transmisión de alertas a Azure Sentinel](../../sentinel/connect-azure-security-center.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="106-automate-the-response-to-security-alerts"></a>10.6: Automatización de la respuesta a las alertas de seguridad

**Guía**: Use la característica Automatización de flujo de trabajo de Azure Security Center para desencadenar automáticamente las respuestas a través de Logic Apps en las alertas y recomendaciones de seguridad para proteger los recursos de Azure.
    

- [Automatización de respuestas a desencadenadores de Security Center](../../security-center/workflow-automation.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="penetration-tests-and-red-team-exercises"></a>Pruebas de penetración y ejercicios del equipo rojo

*Para más información, consulte [Azure Security Benchmark: Pruebas de penetración y ejercicios del equipo rojo](../../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1: Realice pruebas de penetración periódicas de los recursos de Azure y asegúrese de corregir todos los resultados de seguridad críticos

**Guía**: Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Use la estrategia de Microsoft y la ejecución de pruebas de penetración de sitios activos y ataques simulados en la infraestructura en la nube, los servicios y las aplicaciones administrados por Microsoft.

- [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [Información general sobre Azure Security Benchmark V2](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
