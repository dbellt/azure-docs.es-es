---
title: Base de referencia de seguridad de Azure para Azure Kubernetes Service
description: La base de referencia de seguridad de Azure Kubernetes Service proporciona una guía de procedimientos y recursos para implementar las recomendaciones de seguridad especificadas en Azure Security Benchmark.
author: msmbaldwin
ms.service: container-service
ms.topic: conceptual
ms.date: 03/30/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark, devx-track-azurepowershell
ms.openlocfilehash: 0564f1f39ac9d492dfffdf0e7adacdde08db0874
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107769596"
---
# <a name="azure-security-baseline-for-azure-kubernetes-service"></a>Base de referencia de seguridad de Azure para Azure Kubernetes Service

Esta base de referencia de seguridad aplica la guía de la [versión 1.0 de Azure Security Benchmark](../security/benchmarks/overview-v1.md) a Azure Kubernetes. Azure Security Benchmark proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. El contenido se agrupa por los **controles de seguridad** definidos por Azure Security Benchmark y las instrucciones relacionadas aplicables a Azure Kubernetes. Se han excluido los **controles** que no son aplicables a Azure Kubernetes o cuya responsabilidad es de Microsoft.

Para ver cómo Azure Kubernetes se asigna por completo a Azure Security Benchmark, consulte el [archivo de asignación de base de referencia de seguridad de Azure Kubernetes completo](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Seguridad de redes

*Para más información, consulte [Azure Security Benchmark: seguridad de red](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1: Protección de los recursos de Azure dentro de las redes virtuales

**Guía**: De forma predeterminada, un grupo de seguridad de red y una tabla de rutas se crean automáticamente al crear un clúster de Microsoft Azure Kubernetes Service (AKS). AKS modifica automáticamente los grupos de seguridad de red para que el tráfico adecuado fluya a medida que se crean servicios con equilibradores de carga, asignaciones de puertos o rutas de entrada El grupo de seguridad de red se asocia automáticamente con los adaptadores de red virtuales en los nodos de cliente y la tabla de rutas con la subred en la red virtual. 

Use las directivas de red de AKS para limitar el tráfico de red mediante la definición de reglas para el tráfico de entrada y salida entre los pods de Linux en un clúster según la elección de los espacios de nombres y los selectores de etiquetas. El uso de directivas de red requiere el complemento de Azure CNI con redes virtuales y subredes definidas y solo se puede habilitar al crear clústeres. No se pueden implementar en un clúster de AKS existente.

Puede implementar un clúster de AKS privado para asegurarse de que el tráfico de red entre el servidor de la API de AKS y los grupos de nodos solo se mantiene en la red privada. El plano de control o el servidor de la API reside en una suscripción a Azure administrada por AKS y usa direcciones IP internas (RFC1918), mientras que el grupo de clústeres o nodos del cliente se encuentra en su propia suscripción. El servidor y el grupo de clústeres o nodos se comunican entre sí mediante el servicio de Azure Private Link en la red virtual del servidor de la API y un punto de conexión privado expuesto en la subred del clúster de AKS del cliente.  También puede usar un punto de conexión público para el servidor de la API de AKS, pero restringir el acceso con la característica de intervalos IP autorizados del servidor de la API de AKS. 

- [Conceptos de seguridad de las aplicaciones y los clústeres en Azure Kubernetes Service (AKS)](concepts-security.md)

- [Protección del tráfico entre pods mediante directivas de red en Azure Kubernetes Service (AKS)](use-network-policies.md)

- [Creación de un clúster privado de Azure Kubernetes Service](private-clusters.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy - Microsoft.ContainerService**:

[!INCLUDE [Resource Policy for Microsoft.ContainerService 1.1](../../includes/policy/standards/asb/rp-controls/microsoft.containerservice-1-1.md)]

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2: Supervisión y registro de la configuración y el tráfico de redes virtuales, subredes y NIC

**Guía**: Use Security Center y siga sus recomendaciones de protección de red a fin de proteger los recursos de red que usan los clústeres de Azure Kubernetes Service (AKS). 

Habilite los registros de flujo de los grupos de seguridad de red y envíe los registros a una cuenta de Azure Storage para su auditoría. También puede enviar los registros de flujo a un área de trabajo de Log Analytics y, a continuación, usar el Análisis de tráfico para proporcionar información sobre los patrones de tráfico de la nube de Azure y visualizar la actividad de la red, identificar las zonas activas y las amenazas de seguridad, comprender los patrones de flujo de tráfico y detectar los errores de configuración de la red.

- [Descripción de la seguridad de red proporcionada por Azure Security Center](../security-center/security-center-network-recommendations.md)

- [Habilitación de los registros de flujo de seguridad de la red](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Habilitación y uso del Análisis de tráfico](../network-watcher/traffic-analytics.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="13-protect-critical-web-applications"></a>1.3: Proteja las aplicaciones web críticas

**Guía**: Use Web Application Firewall (WAF) habilitado para Azure Application Gateway delante de un clúster de AKS a fin de proporcionar una capa adicional de seguridad al filtrar el tráfico entrante a sus aplicaciones web. Azure WAF usa un conjunto de reglas, que proporciona Open Web Application Security Project (OWASP), para detectar ataques, como el scripting entre sitios o el envenenamiento de cookies en este tráfico. 

Use una puerta de enlace de API para la autenticación, la autorización, la limitación, el almacenamiento en caché, la transformación y la supervisión de las API que se usan en el entorno de AKS. Una puerta de enlace de API actúa como puerta principal de los microservicios, desacopla los clientes de los microservicios y reduce la complejidad de los microservicios al eliminar la carga que supone el control de las cuestiones transversales.

- [Descripción de los procedimientos recomendados con la conectividad de red y la seguridad en AKS](operator-best-practices-network.md)

- [Controlador de entrada de Application Gateway ](../application-gateway/ingress-controller-overview.md)

- [Uso de Azure API Management con microservicios implementados en Azure Kubernetes Service](../api-management/api-management-kubernetes.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4: Deniegue las comunicaciones con direcciones IP malintencionadas conocidas

**Guía**: Habilite la protección estándar de denegación de servicio distribuido (DDoS) de Microsoft en las redes virtuales en las que se implementan los componentes de Azure Kubernetes Service (AKS) para la protección contra ataques DDoS.

Instale el motor de directiva de red y cree directivas de red de Kubernetes para controlar el flujo de tráfico entre pods en AKS a medida que, de manera predeterminada, se permite todo el tráfico entre estos pods. La directiva de red solo se debe usar para los pods y los nodos basados en Linux de AKS. Para mejorar la seguridad, defina reglas que limiten la comunicación del pod. 

Opte por permitir o denegar el tráfico según la configuración, como, por ejemplo, etiquetas asignadas, espacio de nombres o puerto de tráfico. Dado que los pods se crean dinámicamente en un clúster de AKS, se pueden aplicar automáticamente las directivas de red necesarias. 

- [Protección del tráfico entre pods mediante directivas de red en Azure Kubernetes Service (AKS)](use-network-policies.md)

- [Configuración de la protección contra DDoS](../ddos-protection/manage-ddos-protection.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="15-record-network-packets"></a>1.5: Registro de los paquetes de red

**Guía**: Use la captura de paquetes de Network Watcher según sea necesario para investigar actividades anómalas. 

Network Watcher se habilitará automáticamente en la región de la red virtual al crear o actualizar una red virtual en su suscripción. También puede crear nuevas instancias de Network Watcher con PowerShell, la CLI de Azure, la API REST o el método del cliente de Azure Resource Manager.

- [Habilitación de Network Watcher](../network-watcher/network-watcher-create.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6: Implementación de sistemas de prevención de intrusiones y detección de intrusiones (IDS/IPS) basados en la red

**Guía**: Proteja su clúster de Azure Kubernetes Service (AKS) con una instancia de Web Application Firewall (WAF) habilitada para Azure Application Gateway 

Si la detección de intrusiones o la prevención basada en la inspección de la carga o el análisis del comportamiento no se consideran un requisito, se puede usar y configurar una instancia de Azure Application Gateway con WAF en "modo de detección" para registrar alertas y amenazas, o en "modo de prevención" para bloquear activamente las intrusiones y los ataques detectados.

- [Descripción de los procedimientos recomendados para proteger el clúster de AKS con WAF](https://docs.microsoft.com/azure/aks/operator-best-practices-network#secure-traffic-with-a-web-application-firewall-waf)

- [Implementación de Azure Application Gateway (Azure WAF)](../web-application-firewall/ag/application-gateway-web-application-firewall-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8: Minimice la complejidad y la sobrecarga administrativa de las reglas de seguridad de red

**Guía**: use etiquetas de servicio de red virtual para definir controles de acceso a la red en los grupos de seguridad de red asociados a instancias de Azure Kubernetes Service. Se pueden usar etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad a fin de permitir o denegar el tráfico para el servicio correspondiente si se especifica el nombre de la etiqueta de servicio. 

Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian.

Aplique una etiqueta de Azure a grupos de nodos de su clúster de AKS. Estas son diferentes de las etiquetas de servicio de red virtual, ya que se aplican a cada nodo dentro del grupo de nodos y se conservan a lo largo de las actualizaciones. 

- [Descripción y uso de etiquetas de servicio](../virtual-network/service-tags-overview.md)

- [Descripción de grupos de seguridad de red para AKS](support-policies.md)

- [Control del tráfico de salida de los nodos de clúster en Azure Kubernetes Service (AKS)](limit-egress-traffic.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9: Mantenga las configuraciones de seguridad estándar para dispositivos de red

**Guía**: defina e implemente configuraciones de seguridad estándar con Azure Policy para los recursos de red asociados a los clústeres de Azure Kubernetes Service (AKS). 

Utilice alias de Azure Policy en los espacios de nombres "Microsoft.ContainerService" y "Microsoft.Network" para crear directivas personalizadas que permitan auditar o aplicar la configuración de red de los clústeres de AKS. 

Además, use definiciones de directivas integradas relacionadas con AKS, como las siguientes:

- Los intervalos IP autorizados deben definirse en los servicios de Kubernetes

- Exigir la entrada de HTTPS en el clúster de Kubernetes

- Asegurarse de que los servicios solo escuchan en los puertos permitidos en el clúster de Kubernetes

Información adicional disponible en los vínculos indicados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Ejemplos de Azure Policy para redes](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#network)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="110-document-traffic-configuration-rules"></a>1.10: Documente las reglas de configuración de tráfico

**Guía**: use etiquetas para los grupos de seguridad de red y otros recursos para el flujo de tráfico hacia y desde clústeres de Azure Kubernetes Service (AKS). Use el campo "Descripción" para las reglas del grupo de seguridad de red a fin de especificar la duración o la necesidad del negocio y otra información de cualquier regla que permita el tráfico hacia o desde una red.

Use cualquiera de las definiciones integradas relacionadas con el etiquetado de Azure Policy, como "Requerir etiqueta y su valor", que garanticen que todos los recursos se crean con etiquetas y para recibir notificaciones para los recursos no etiquetados existentes.

Opte por permitir o denegar rutas de acceso de red específicas en el clúster en función de los espacios de nombres y los selectores de etiquetas con las directivas de red. Utilice estos espacios de nombres y estas etiquetas como descriptores para las reglas de configuración del tráfico. Use Azure PowerShell o la interfaz de la línea de comandos (CLI) de Azure para buscar los recursos o realizar acciones en ellos en función de sus etiquetas.

- [Azure Policy con la CLI](/cli/azure/policy)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

- [Creación de un grupo de seguridad de red con una configuración de seguridad](../virtual-network/tutorial-filter-network-traffic.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11: Use herramientas automatizadas para supervisar las configuraciones de recursos de red y detectar cambios

**Guía**: Use el registro de actividad de Azure para supervisar las configuraciones de los recursos de red y detectar cambios en los recursos de red relacionados con los clústeres de Azure Kubernetes Service (AKS). 

Cree alertas en Azure Monitor que se desencadenarán cuando se produzcan cambios en los recursos de red críticos. Las entradas del usuario AzureContainerService en los registros de actividad se registran como acciones de plataforma. 

Use los registros de Azure Monitor para habilitar y consultar los registros de AKS, los componentes maestros, kube-apiserver y kube-controller-manager. Cree y administre los nodos que ejecutan kubelet con el entorno de ejecución del contenedor, e implemente las aplicaciones mediante el servidor de API de Kubernetes administrado. 

- [Visualización y recuperación de eventos del registro de actividad de Azure](/azure/azure-monitor/platform/activity-log#view-the-activity-log)

- [Creación de alertas en Azure Monitor](/azure/azure-monitor/platform/alerts-activity-log)

- [Habilitación y revisión de los registros del nodo maestro de Kubernetes en Azure Kubernetes Service (AKS)](/azure/aks/view-master-logs)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="logging-and-monitoring"></a>Registro y supervisión

*Para más información, consulte [Azure Security Benchmark: registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1: Uso de orígenes de sincronización de hora aprobados

**Guía**: Los nodos de Azure Kubernetes Service (AKS) usan ntp.ubuntu.com para la sincronización de hora, junto con el puerto UDP 123 y el protocolo de tiempo de red (NTP). 

Asegúrese de que los nodos de clúster pueden acceder a los servidores NTP si usa servidores DNS personalizados. 

- [Descripción de los requisitos de dominio y puerto de NTP para los nodos de clúster de AKS](limit-egress-traffic.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="22-configure-central-security-log-management"></a>2.2: Configuración de la administración central de registros de seguridad

**Guía**: Habilite los registro de auditoría de los componentes principales de Azure Kubernetes Service (AKS), kube-apiserver y kube-controller-manager, que se proporcionan como un servicio administrado. 

- kube-auditaksService: el nombre para mostrar en el registro de auditoría de la operación de plano de control (desde hcpService). 

- masterclient: el nombre para mostrar en el registro de auditoría de MasterClientCertificate, el certificado que obtiene desde az aks get-credentials. 

- nodeclient: el nombre para mostrar de ClientCertificate, que los nodos de agente usan.

Habilite también otros registros de auditoría como kube-audit. 

Exporte estos registros a Log Analytics u otra plataforma de almacenamiento. En Azure Monitor, use áreas de trabajo de Log Analytics para realizar consultas y análisis, y use cuentas de Azure Storage para el almacenamiento de archivos a largo plazo.

Habilite los datos integrados en Azure Sentinel o en una SIEM de terceros, en función de los requisitos empresariales de la organización.

- [Revisión del esquema de registro, incluidos los roles de registro](/azure/aks/view-master-logs)

- [Descripción de Azure Monitor para contenedores](/azure/azure-monitor/insights/container-insights-overview)

- [Habilitación de Azure Monitor para contenedores](/azure/azure-monitor/insights/container-insights-onboard)

- [Habilitación y revisión de los registros del nodo maestro de Kubernetes en Azure Kubernetes Service (AKS)](/azure/aks/view-master-logs)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3: Habilitación del registro de auditoría para recursos de Azure

**Guía**: use los registros de actividad para supervisar las acciones en los recursos de Azure Kubernetes Service (AKS) y ver toda la actividad y su estado. Determine qué operaciones se efectuaron en los recursos de la suscripción con los registros de actividad: 

- quién inició la operación
- cuándo tuvo lugar la operación
- el estado de la operación
- los valores de otras propiedades que podrían ayudarle en la investigación de la operación

puede recuperar información de los registros de actividad mediante Azure PowerShell, la interfaz de la línea de comandos (CLI) de Azure, la API REST de Azure o Azure Portal. 

Habilite los registros de auditoría en los componentes maestros de AKS, como: 

- kube-auditaksService: el nombre para mostrar en el registro de auditoría de la operación de plano de control (desde hcpService). 

- masterclient: el nombre para mostrar en el registro de auditoría de MasterClientCertificate, el certificado que obtiene desde az aks get-credentials. 

- nodeclient: el nombre para mostrar de ClientCertificate, que los nodos de agente usan.

Habilite también otros registros de auditoría, como kube-audit. 

- [Habilitación y revisión de los registros del nodo maestro de Kubernetes en AKS](/azure/aks/view-master-logs)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4: Recopilación de registros de seguridad de sistemas operativos

**Guía**: habilite la instalación automática de los agentes de Log Analytics para recopilar datos de los nodos de clúster de AKS. Además, active el aprovisionamiento automático del agente de supervisión de Azure Log Analytics desde Azure Security Center, dado que, de forma predeterminada, el aprovisionamiento automático está desactivado. El agente también se puede instalar manualmente. Con el aprovisionamiento automático activado, Security Center implementa el agente de Log Analytics en todas las máquinas virtuales de Azure compatibles y en las nuevas que se crean. Security Center recopila datos de Azure Virtual Machines, los conjuntos de escalado de máquinas virtuales y los contenedores de IaaS (como los nodos de clúster de Kubernetes) a fin de supervisar las amenazas y vulnerabilidades de seguridad. Los datos se recopilan con el agente de Azure Log Analytics, que lee distintas configuraciones relacionadas con la seguridad y distintos registros de eventos de la máquina y copia los datos en el área de trabajo para analizarlos. 

La recopilación de datos es necesaria para proporcionar visibilidad sobre actualizaciones que faltan, valores de seguridad del sistema operativo mal configurados, estado de la protección de punto de conexión y detecciones de amenazas y mantenimiento.

- [Habilitación del aprovisionamiento automático del agente de Log Analytics](../security-center/security-center-enable-data-collection.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="25-configure-security-log-storage-retention"></a>2.5: Configuración de la retención del almacenamiento de registros de seguridad

**Guía**: Incorpore las instancias de Azure Kubernetes Service (AKS) a Azure Monitor y establezca el período de retención del área de trabajo de Azure Log Analytics correspondiente según los requisitos de cumplimiento de su organización. 

- [Configuración de parámetros de retención de registros de áreas de trabajo de Log Analytics](/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="26-monitor-and-review-logs"></a>2.6: Supervisión y registros de revisión

**Guía**: Incorpore las instancias de Azure Kubernetes Service (AKS) para Azure Monitor y establezca la configuración de diagnóstico para el clúster. 

Use el área de trabajo de Log Analytics de Azure Monitor para revisar los registros y realizar consultas en los datos del registro. Los registros de Azure Monitor se habilitan y administran en Azure Portal o a través de la CLI y funcionan con los clústeres de control de acceso basado en rol de Kubernetes (RBAC de Kubernetes) y los de AKS que no están habilitados para RBAC.

Vea los registros que generan los componentes maestros de AKS (kube-apiserver y kube-controllermanager) para solucionar problemas de la aplicación y los servicios. Habilite e incorpore datos en Azure Sentinel o en un SIEM de terceros para la administración y supervisión de registros centralizadas.

- [Habilitación y revisión de los registros del nodo maestro de Kubernetes en AKS](/azure/aks/view-master-logs)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Procedimiento para realizar consultas personalizadas en Azure Monitor](/azure/azure-monitor/log-query/get-started-queries)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7: Habilitación de alertas para actividades anómalas

**Guía**: Use Azure Kubernetes Service (AKS) junto con Security Center para obtener una visibilidad más profunda de los nodos de AKS. 

Revise las alertas de Security Center sobre amenazas y actividades malintencionadas detectadas en el host y a nivel de clúster. Security Center implementa análisis continuos de eventos de seguridad sin procesar que se producen en un clúster de AKS, como datos de red, la creación de procesos y el registro de auditoría de Kubernetes. Determine si esta actividad es el comportamiento esperado o si la aplicación no se comporta correctamente. Use las métricas y los registros de Azure Monitor para apoyar sus conclusiones. 

- [Descripción de la integración de Azure Kubernetes Services con Security Center](../security-center/defender-for-kubernetes-introduction.md)

- [Habilitación del nivel estándar de Azure Security Center](../security-center/security-center-get-started.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="28-centralize-anti-malware-logging"></a>2.8: Centralización del registro antimalware

**Guía**: Instale y habilite Microsoft Antimalware para Azure en máquinas virtuales de Azure Kubernetes Service (AKS) y nodos del conjunto de escalado de máquinas virtuales. Revise las alertas en Security Center para su corrección.

- [Microsoft Antimalware para Azure Cloud Services y Virtual Machines](../security/fundamentals/antimalware.md)

- [Guía de referencia de las alertas de seguridad](../security-center/alerts-reference.md)

- [Alertas para contenedores: clústeres de Azure Kubernetes Service](https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-akscluster)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="29-enable-dns-query-logging"></a>2.9: Habilitación del registro de consultas DNS

**Guía**: Azure Kubernetes Service (AKS) usa el proyecto CoreDNS para la administración y la resolución DNS del clúster.

Para habilitar el registro de consultas de DNS, aplique la configuración documentada en coredns-custom ConfigMap. 

- [Personalización de CoreDNS con Azure Kubernetes Service](coredns-custom.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="210-enable-command-line-audit-logging"></a>2.10: Habilitación del registro de auditoría de la línea de comandos

**Guía**: Use kubectl, un cliente de línea de comandos, en Azure Kubernetes Service (AKS) para administrar un clúster de Kubernetes y obtener sus registros del nodo de AKS a fin de solucionar problemas. Si usa Azure Cloud Shell, kubectl ya está instalado. Para instalar kubectl localmente, use el cmdlet "Install-AzAksKubectl".

- [Inicio rápido: implementación de un clúster de Azure Kubernetes Service mediante PowerShell](kubernetes-walkthrough-powershell.md)

- [Obtención de registros de kubelet desde nodos de clúster de Azure Kubernetes Service (AKS)](kubelet-logs.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="identity-and-access-control"></a>Identidad y Access Control

*Para más información, consulte [Azure Security Benchmark: identidad y control de acceso](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1: Mantenga un inventario de cuentas administrativas

**Guía**: Azure Kubernetes Service (AKS) por sí solo no proporciona una solución de administración de identidades en la que se almacenen contraseñas y cuentas de usuario normales. Con la integración de Azure Active Directory (Azure AD), puede conceder a los usuarios o grupos acceso a los recursos de Kubernetes de un espacio de nombres o del clúster.

Realice consultas ad hoc a fin de detectar cuentas que son miembros de grupos administrativos de AKS con el módulo de PowerShell de Azure AD.

Use la CLI de Azure para operaciones como "Get access credentials for a managed Kubernetes cluster" ("Obtener credenciales de acceso para un clúster de Kubernetes administrado") a fin de ayudar a conciliar el acceso de forma periódica. Implemente este proceso para mantener un inventario actualizado de las cuentas de servicio, que son otro tipo de usuario principal en AKS. Aplique las recomendaciones de administración de identidad y acceso de Security Center.

- [Integración de AKS con Azure AD](azure-ad-integration-cli.md)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

- [Supervisión de identidad y acceso con Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="32-change-default-passwords-where-applicable"></a>3.2: Cambie las contraseñas predeterminadas cuando proceda

**Guía**: Azure Kubernetes Service (AKS) no tiene el concepto de contraseñas predeterminadas comunes y no proporciona una solución de administración de identidades en la que se puedan almacenar las contraseñas y las cuentas de usuario normales. Con la integración de Azure Active Directory (Azure AD), puede conceder acceso basado en rol a los recursos de Kubernetes de un espacio de nombres o del clúster. 

Realice consultas ad hoc a fin de detectar cuentas que son miembros de grupos administrativos de AKS con el módulo de PowerShell de Azure AD.

- [Descripción de las opciones de acceso e identidad de AKS](concepts-identity.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="33-use-dedicated-administrative-accounts"></a>3.3: Use cuentas administrativas dedicadas

**Guía**: Integre la autenticación de usuario para los clústeres de Azure Kubernetes Service (AKS) con Azure Active Directory (Azure AD). Inicie sesión en un clúster de AKS mediante un token de autenticación de Azure AD. Configure el control de acceso basado en rol de Kubernetes (Kubernetes RBAC) para habilitar el acceso administrativo a información y permisos de la configuración de Kubernetes (kubeconfig), espacios de nombres y recursos de clúster. 

Cree directivas y procedimientos en torno al uso de cuentas administrativas dedicadas. Implemente las recomendaciones de administración de identidad y acceso de Security Center.

- [Supervisión de identidad y acceso con Azure Security Center](../security-center/security-center-identity-access.md)

- [Control del acceso a los recursos de clúster](azure-ad-rbac.md)

- [Uso de los controles de acceso basados en rol](control-kubeconfig-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4: Use el inicio de sesión único (SSO) con Azure Active Directory

**Guía**: Use el inicio de sesión único para Azure Kubernetes Service (AKS) con la autenticación integrada de Azure Active Directory (Azure AD) de un clúster de AKS.

- [Visualización de los registros de Kubernetes, los eventos y las métricas de pods en tiempo real](/azure/azure-monitor/insights/container-insights-livedata-overview)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5: Use la autenticación multifactor para todo el acceso basado en Azure Active Directory

**Guía**: Integre la autenticación para Azure Kubernetes Service (AKS) con Azure Active Directory (Azure AD). 

Habilite la autenticación multifactor de Azure AD y siga las recomendaciones de administración de identidades y acceso de Security Center.

- [Habilitación de la autenticación multifactor en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Supervisión de la identidad y el acceso en Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6: Use máquinas dedicadas (estaciones de trabajo de acceso con privilegios) para todas las tareas administrativas

**Guía**: Use una estación de trabajo de acceso con privilegios (PAW), con Multi-Factor Authentication (MFA), configurada para iniciar sesión en los clústeres de Azure Kubernetes Service especificados y los recursos relacionados.

- [Más información sobre las estaciones de trabajo con privilegios de acceso](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Cómo habilitar la autenticación multifactor (MFA) en Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7: Registro y alerta de actividades sospechosas desde cuentas administrativas

**Guía**: use los informes de seguridad de Azure Active Directory (Azure AD) con la autenticación integrada de Azure AD para Azure Kubernetes Service (AKS). Se pueden generar alertas cuando se producen actividades sospechosas o no seguras en el entorno. Use Security Center para supervisar la actividad de identidad y acceso.

- [Procedimiento para identificar usuarios de Azure AD marcados por una actividad de riesgo](../active-directory/identity-protection/overview-identity-protection.md)

- [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="38-manage-azure-resources-only-from-approved-locations"></a>3.8: Administración de los recursos de Azure solo desde ubicaciones aprobadas

**Guía**: use ubicaciones con nombre de acceso condicional para permitir el acceso a los clústeres de Azure Kubernetes Service (AKS) solo en agrupaciones lógicas específicas de intervalos de direcciones IP o países o regiones. Requiere la autenticación integrada para AKS con Azure Active Directory (Azure AD).

Limite el acceso al servidor de la API de AKS desde un conjunto limitado de intervalos de direcciones IP, a medida que recibe solicitudes para realizar acciones en el clúster con el fin de crear recursos o escalar el número de nodos. 

- [Protección del acceso al servidor de API con intervalos de direcciones IP autorizadas en Azure Kubernetes Service (AKS)](api-server-authorized-ip-ranges.md)

- [Configuración de ubicaciones con nombre en Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="39-use-azure-active-directory"></a>3.9: Uso de Azure Active Directory

**Guía**: use Azure Active Directory (Azure AD) como sistema central de autenticación y autorización para Azure Kubernetes Service (AKS). Azure AD protege los datos mediante el cifrado seguro para datos en reposo y en tránsito, y también cifra con sal las credenciales de usuario, les aplica hash y las almacena de forma segura.

Use los roles integrados de AKS con el control de acceso basado en rol de Azure (RBAC de Azure) (colaborador y propietario de directivas de recursos) para las operaciones de asignación de directivas en el clúster de Kubernetes.

- [Introducción a Azure Policy](../governance/policy/overview.md)

- [Integración de Azure AD con AKS](azure-ad-integration-cli.md)

- [Integración de Azure AD administrado por AKS](managed-aad.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10: Revise y concilie regularmente el acceso de los usuarios

**Guía**: use los informes de seguridad de Azure Active Directory (Azure AD) con la autenticación integrada de Azure AD para Azure Kubernetes Service (AKS). Busque registros de Azure AD para detectar cuentas obsoletas. 

Realice revisiones de acceso de identidad de Azure para administrar de forma eficiente las pertenencias a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles. Corrija las recomendaciones de identidad y acceso de Security Center.

Tenga en cuenta los roles que se usan para la compatibilidad o con fines de solución de problemas. Por ejemplo, las acciones de clúster que realiza el servicio de soporte técnico de Microsoft (con el consentimiento del usuario) se realizan en el rol "editar" integrado de Kubernetes que se denomina aks-support-rolebinding. La compatibilidad con AKS está habilitada con este rol para editar la configuración y los recursos del clúster a fin de solucionar y diagnosticar problemas del clúster. Sin embargo, este rol no puede modificar permisos ni crear roles ni enlaces de roles. El acceso a roles solo se habilita en incidencias de soporte técnico activas con acceso Just-in-Time (JIT).
 
- [Opciones de acceso e identidad en Azure Kubernetes Service (AKS)](concepts-identity.md)

- [Procedimiento para usar las revisiones de acceso de identidad de Azure](../active-directory/governance/access-reviews-overview.md)

- [Supervisión de la actividad de identidad y acceso de los usuarios en Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11: Supervisión de los intentos de acceso a credenciales desactivadas

**Guía**: Integre la autenticación del usuario para Azure Kubernetes Service (AKS) con Azure Active Directory (Azure AD). Cree una configuración de diagnóstico para Azure AD y envíe los registros de auditoría y de inicio de sesión a un área de trabajo de Azure Log Analytics. Configure las alertas que quiera (por ejemplo, cuando una cuenta desactivada intente iniciar sesión) de un área de trabajo de Azure Log Analytics.
- [Integración de los registros de actividad de Azure en Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Creación, visualización y administración de alertas de registro mediante Azure Monitor](/azure/azure-monitor/platform/alerts-log)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12: Alerte de las desviaciones de comportamiento en los inicios de sesión de las cuentas

**Guía**: Integre la autenticación del usuario para Azure Kubernetes Service (AKS) con Azure Active Directory (Azure AD). Use las características de protección de identidad y detección de riesgo de Azure AD para configurar respuestas automatizadas a las acciones sospechosas detectadas relacionadas con las identidades de los usuarios. Ingiera datos en Azure Sentinel para investigarlos más según las necesidades empresariales.

- [Visualización de los inicios de sesión de riesgo de Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Configuración y habilitación de las directivas de riesgo de protección de identidad](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Incorporación de Azure Sentinel](../sentinel/quickstart-onboard.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="data-protection"></a>Protección de datos

*Para más información, consulte [Azure Security Benchmark: protección de datos](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1: Mantenimiento de un inventario de información confidencial

**Guía**: use etiquetas en los recursos relacionados con las implementaciones de Azure Kubernetes Service para ayudar a realizar un seguimiento de los recursos de Azure que almacenan o procesan información confidencial.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

- [Actualización de etiquetas para clústeres administrados](/rest/api/aks/managedclusters/updatetags)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2: Aislamiento de los sistemas que almacenan o procesan información confidencial

**Guía**: Aísle lógicamente los equipos y las cargas de trabajo en el mismo clúster con Azure Kubernetes Service (AKS) para proporcionar el número mínimo de privilegios, en el ámbito de los recursos necesarios para cada equipo. 

Use el espacio de nombres en Kubernetes para crear un límite de aislamiento lógico. Considere la posibilidad de implementar características adicionales de Kubernetes para el aislamiento y la opción multiinquilino, como la programación, las redes, la autenticación, la autorización y los contenedores.

Implemente suscripciones o grupos de administración independientes para los entornos de desarrollo, pruebas y producción. Separe los clústeres de AKS con las redes mediante su implementación en distintas redes virtuales, que se etiquetan adecuadamente.

- [Información sobre los procedimientos recomendados para el aislamiento de clústeres en AKS](operator-best-practices-cluster-isolation.md)

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Descripción de los procedimientos recomendados con la conectividad de red y la seguridad en AKS](operator-best-practices-network.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3: Supervisión y bloqueo de una transferencia no autorizada de información confidencial

**Guía**: Use una solución de terceros de Azure Marketplace en los perímetros de red que supervise la transferencia no autorizada de información confidencial y bloquee dichas transferencias, a la vez que alerta a los profesionales de seguridad de la información.

Microsoft administra la plataforma subyacente, trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

- [Lista de puertos, direcciones y nombres de dominio necesarios para la funcionalidad de AKS](limit-egress-traffic.md)

- [Configuración de los valores de diagnóstico para Azure Firewall](../firewall/firewall-diagnostics.md)

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4: Cifrado de toda la información confidencial en tránsito

**Guía**: Cree un controlador de entrada HTTPS y use sus propios certificados TLS (o bien, de Let's Encrypt) para sus implementaciones de Azure Kubernetes Service (AKS). 

El tráfico de salida de Kubernetes se cifra de forma predeterminada a través de HTTPS/TLS. Revise cualquier tráfico de salida potencialmente no cifrado de las instancias de AKS para realizar una supervisión adicional. Este puede incluir tráfico NTP, DNS o HTTP para recuperar actualizaciones en algunos casos. 

- [Creación de un controlador de entrada HTTPS en AKS y uso de sus propios certificados TLS](ingress-own-tls.md)

- [Creación de un controlador de entrada HTTPS en AKS con Let's Encrypt](ingress-tls.md)

- [Lista de los posibles puertos y protocolos salientes usados por AKS](limit-egress-traffic.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5: Uso de una herramienta de detección activa para identificar datos confidenciales

**Instrucciones**: las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Storage ni los recursos de proceso. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.
Microsoft administra la plataforma subyacente, trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. 

Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4.6: Uso de RBAC de Azure para administrar el acceso a los recursos

**Guía**: Use el sistema de autorización del control de acceso basado en rol de Azure (RBAC de Azure) que se basa en Azure Resource Manager a fin de proporcionar una administración de acceso específico de los recursos de Azure.

Configure Azure Kubernetes Service (AKS) para que utilice Azure Active Directory (Azure AD) para la autenticación de usuarios. Inicie sesión en un clúster de AKS mediante un token de autenticación de Azure AD con esta configuración. 

Use los roles integrados de AKS con RBAC de Azure (colaborador y propietario de directivas de recursos) para las operaciones de asignación de directivas en el clúster de AKS.

- [Control del acceso a los recursos de clúster mediante RBAC de Azure y las identidades de Azure AD en AKS](azure-ad-rbac.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy - Microsoft.ContainerService**:

[!INCLUDE [Resource Policy for Microsoft.ContainerService 4.6](../../includes/policy/standards/asb/rp-controls/microsoft.containerservice-4-6.md)]

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7: Uso de la prevención de pérdida de datos basada en host para aplicar el control de acceso

**Guía**: las características de identificación, clasificación y prevención de pérdida de datos todavía no están disponibles para Azure Storage ni los recursos de proceso. Implemente una solución de terceros, si es necesario, para fines de cumplimiento.
Microsoft administra la plataforma subyacente, trata todo el contenido de los clientes como confidencial y hace grandes esfuerzos para proteger a los clientes contra la pérdida y exposición de sus datos. Para garantizar la seguridad de los datos de los clientes dentro de Azure, Microsoft ha implementado y mantiene un conjunto de controles y funcionalidades eficaces de protección de datos.

- [Descripción de la protección de datos de los clientes en Azure](../security/fundamentals/protection-customer-data.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8: Cifrado de información confidencial en reposo

**Guía**: Los dos tipos principales de almacenamiento proporcionados para volúmenes en Azure Kubernetes Service (AKS) están respaldados por Azure Disks o Azure Files. A fin de mejorar la seguridad, ambos tipos de almacenamiento usan Azure Storage Service Encryption (SSE) para cifrar los datos en reposo. De manera predeterminada, los datos se cifran con claves administradas por Microsoft.

El cifrado en reposo mediante claves administradas por el cliente está disponible para el cifrado de los discos de datos y del sistema operativo de los clústeres de AKS con tal de tener un mayor control sobre las claves de cifrado. Los clientes son responsables de las actividades de administración de claves, como la rotación o la copia de seguridad de estas. Actualmente no se pueden cifrar los discos con Azure Disk Encryption en el nivel de nodo de AKS.

- [Procedimientos recomendados para el almacenamiento y las copias de seguridad en Azure Kubernetes Service (AKS)](operator-best-practices-storage.md)

- [Traiga sus propias claves (BYOK) con discos de Azure en Azure Kubernetes Service (AKS)](azure-disk-customer-managed-keys.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9: Registro y alerta de cambios en los recursos críticos de Azure

**Guía**: Use Azure Monitor para contenedores a fin de supervisar el rendimiento de las cargas de trabajo de contenedor implementadas en clústeres de Kubernetes administrados y hospedados en Azure Kubernetes Service (AKS). 

Configure alertas para recibir notificaciones de manera proactiva o crear registros cuando el uso de la CPU y la memoria en nodos o contenedores supere los umbrales, o cuando se produzca un cambio en el estado de mantenimiento del clúster en la infraestructura o la acumulación de estado de los nodos. 

Use el registro de actividad de Azure para supervisar los clústeres de AKS y los recursos relacionados en un nivel alto. Integrarse con Prometheus para ver las métricas de la aplicación y de la carga de trabajo que recopila de los nodos y Kubernetes mediante consultas para crear alertas personalizadas y paneles y realizar un análisis detallado.

- [Descripción de Azure Monitor para contenedores](/azure/azure-monitor/insights/container-insights-overview)

- [Cómo habilitar Azure Monitor para contenedores](/azure/azure-monitor/insights/container-insights-onboard)

- [Visualización y recuperación de eventos del registro de actividad de Azure](/azure/azure-monitor/platform/activity-log#view-the-activity-log)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="vulnerability-management"></a>Administración de vulnerabilidades

*Para más información, consulte [Azure Security Benchmark: administración de vulnerabilidades](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1: Ejecute herramientas de análisis de vulnerabilidades automatizado

**Guía**: Use Security Center para supervisar su instancia de Azure Container Registry, incluidas las instancias de Azure Kubernetes Service (AKS) con tal de detectar vulnerabilidades. Habilite la agrupación de registros de contenedor en Security Center para asegurarse de que dicho sistema esté listo para examinar las imágenes que se inserten en el registro.

Reciba una notificación en el panel de Security Center cuando se detecten problemas después de que Security Center examine la imagen mediante Qualys. La característica de agrupación de registros de contenedor proporciona una visibilidad más detallada de las vulnerabilidades de las imágenes utilizadas en los registros basados en Azure Resource Manager. 

Use Security Center para obtener recomendaciones útiles para cada vulnerabilidad. Estas recomendaciones incluyen una clasificación de gravedad y una guía para la corrección. 

- [Procedimientos recomendados para la administración y la protección de las imágenes de contenedor en Azure Kubernetes Service (AKS)](../security-center/defender-for-container-registries-introduction.md)

- [Procedimientos recomendados para la administración y la seguridad de las imágenes de contenedor en AKS](operator-best-practices-container-image-management.md)

- [Comprensión de la integración de los registros de contenedor con Azure Security Center](../security-center/defender-for-container-registries-introduction.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2: Implemente una solución de administración de revisiones de sistema operativo automatizada

**Guía**: Las actualizaciones de seguridad se aplican automáticamente a los nodos de Linux a fin de proteger los clústeres de Azure Kubernetes Service (AKS) del cliente. Estas actualizaciones incluyen las revisiones de seguridad del sistema operativo o las actualizaciones del kernel. 

Tenga en cuenta que el proceso para mantener actualizados los nodos de Windows Server difiere de los nodos que ejecutan Linux, ya que los nodos de Windows Server no reciben actualizaciones diarias. En su lugar, los clientes deben realizar una actualización en los grupos de nodos de Windows Server de sus clústeres de AKS, que implementa nuevos nodos con la imagen base de Windows Server más reciente y revisiones mediante el panel de control de Azure o la CLI de Azure. Estas actualizaciones contienen mejoras de seguridad o funcionalidad para AKS.

- [Descripción de cómo se aplican las actualizaciones en los nodos de clúster de AKS que ejecutan Linux](node-updates-kured.md)

- [Actualización de un grupo de nodos de AKS para los clústeres de AKS que usan los nodos de Windows Server](https://docs.microsoft.com/azure/aks/use-multiple-node-pools#upgrade-a-node-pool)

- [Actualizaciones de las imágenes de nodos de Azure Kubernetes Service (AKS)](node-image-upgrade.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="53-deploy-an-automated-patch-management-solution-for-third-party-software-titles"></a>5.3: Implementación de una solución de administración de revisiones automatizada de títulos de software de terceros

**Guía**: implemente un proceso manual para asegurarse de que las aplicaciones de terceros del nodo de clúster de Azure Kubernetes Service (AKS) permanecen revisadas durante la vigencia del clúster. Esto puede requerir la habilitación de actualizaciones automáticas, la supervisión de nodos o la realización de reinicios periódicos.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: [Azure Security Benchmark](/azure/governance/policy/samples/azure-security-benchmark) es la iniciativa de directiva predeterminada de Security Center y es la base de sus [recomendaciones](/azure/security-center/security-center-recommendations). Security Center habilita automáticamente las definiciones de Azure Policy relacionadas con este control. Puede que las alertas relacionadas con este control requieran un plan de [Azure Defender](/azure/security-center/azure-defender) para los servicios relacionados.

**Definiciones integradas de Azure Policy - Microsoft.ContainerService**:

[!INCLUDE [Resource Policy for Microsoft.ContainerService 5.3](../../includes/policy/standards/asb/rp-controls/microsoft.containerservice-5-3.md)]

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Compare los exámenes de vulnerabilidades opuestos

**Guía**: Exporte los resultados de exámenes de Security Center en intervalos coherentes y compare los resultados para verificar que se han corregido las vulnerabilidades.

Use el cmdlet de PowerShell "Get-AzSecurityTask" para automatizar la recuperación de tareas de seguridad que Security Center recomienda que realice a fin de reforzar su nivel de seguridad y corregir los resultados del examen de vulnerabilidades.

- [Uso de PowerShell para ver las vulnerabilidades que detecta Azure Security Center](/powershell/module/az.security/get-azsecuritytask)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5: Use un proceso de clasificación de riesgos para priorizar la corrección de las vulnerabilidades detectadas

**Guía**: Use la clasificación de gravedad que proporciona Security Center para dar prioridad a la corrección de vulnerabilidades. 

Use Common Vulnerability Scoring System (CVSS) (u otros sistemas de puntuación que proporciona la herramienta de exploración) si usa una herramienta de evaluación de vulnerabilidades integrada (como Qualys o Rapid7, que ofrece Azure).

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="inventory-and-asset-management"></a>Administración de recursos y del inventario

*Para más información, consulte [Azure Security Benchmark: inventario y administración de recursos](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1: Uso de la solución de detección de recursos automatizada

**Guía**: use Azure Resource Graph para consultar o detectar todos los recursos (por ejemplo, proceso, almacenamiento, red, etc.) de sus suscripciones. Asegúrese de que tiene los permisos adecuados (lectura) en el inquilino y de que puede enumerar todas las suscripciones de Azure, así como los recursos de las suscripciones.

Aunque los recursos clásicos de Azure se pueden detectar a través de Resource Graph, se recomienda encarecidamente crear y usar los recursos basados en Azure Resource Manager en adelante.

- [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

- [Visualización de las suscripciones de Azure](/powershell/module/az.accounts/get-azsubscription)

- [Descripción de Azure RBAC](../role-based-access-control/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="62-maintain-asset-metadata"></a>6.2: Mantenimiento de metadatos de recursos

**Guía**: aplique etiquetas a los recursos de Azure con metadatos para organizarlos de forma lógica en una taxonomía.

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="63-delete-unauthorized-azure-resources"></a>6.3: Eliminación de recursos de Azure no autorizados

**Instrucciones**: Use el etiquetado, los grupos de administración y las suscripciones independientes, si procede, para organizar y realizar un seguimiento de los recursos. 

Aplique taints o etiquetas al crear un grupo de nodos de Azure Kubernetes Service (AKS). Todos los nodos de ese grupo de nodos también heredarán dicho taint o etiqueta.

Los taints o las etiquetas se pueden usar para conciliar el inventario periódicamente y asegurarse de que los recursos no autorizados se eliminan de la suscripción de manera oportuna.

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Creación y uso de etiquetas](../azure-resource-manager/management/tag-resources.md)

- [Clústeres administrados: actualización de etiquetas](/rest/api/aks/managedclusters/updatetags)

- [Especificación de un valor taint o una etiqueta para un grupo de nodos](https://docs.microsoft.com/azure/aks/use-multiple-node-pools#specify-a-taint-label-or-tag-for-a-node-pool)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4: Definición y mantenimiento de un inventario de los recursos de Azure aprobados

**Guía**: defina una lista de recursos de Azure aprobados y el software aprobado para los recursos de proceso según las necesidades organizativas de la empresa.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5: Supervisión de recursos de Azure no aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:
-   Tipos de recursos no permitidos 

-   Tipos de recursos permitidos

Use Azure Resource Graph para consultar o detectar recursos en las suscripciones. Asegúrese de que todos los recursos de Azure del entorno se aprueben según los requisitos organizativos de la empresa.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Creación de consultas con Azure Graph](../governance/resource-graph/first-query-portal.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6: Supervisión de aplicaciones de software no aprobadas en recursos de proceso

**Guía**: Use las características de Seguimiento de cambios e inventario de Azure Automation para averiguar el software que hay instalado en su entorno. 

Recopile y vea el inventario de software, archivos, demonios de Linux, servicios de Windows y claves del Registro de Windows en los equipos y supervise las aplicaciones de software no aprobadas. 

Realice un seguimiento de las configuraciones de sus máquinas a fin de identificar problemas de funcionamiento en el entorno y comprender mejor el estado de las máquinas.

- [Habilitación del inventario de máquinas virtuales de Azure](../automation/automation-tutorial-installed-software.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Eliminación de aplicaciones de software y recursos de Azure no aprobadas

**Guía**: Use las características de Seguimiento de cambios e inventario de Azure Automation para averiguar el software que hay instalado en su entorno. 

Recopile y vea el inventario de software, archivos, demonios de Linux, servicios de Windows y claves del Registro de Windows en los equipos y supervise las aplicaciones de software no aprobadas. 

Realice un seguimiento de las configuraciones de sus máquinas a fin de identificar problemas de funcionamiento en el entorno y comprender mejor el estado de las máquinas.

- [Habilitación del inventario de máquinas virtuales de Azure](../automation/automation-tutorial-installed-software.md)

- [Habilitación de la supervisión de la integridad de los archivos](../security-center/security-center-file-integrity-monitoring.md)

- [Descripción de Azure Change Tracking](../automation/change-tracking/overview.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="68-use-only-approved-applications"></a>6.8: Uso exclusivo de aplicaciones aprobadas

**Guía**: Use las características de Seguimiento de cambios e inventario de Azure Automation para averiguar el software que hay instalado en su entorno. 

Recopile y vea el inventario de software, archivos, demonios de Linux, servicios de Windows y claves del Registro de Windows en los equipos y supervise las aplicaciones de software no aprobadas. 

Realice un seguimiento de las configuraciones de sus máquinas a fin de identificar problemas de funcionamiento en el entorno y comprender mejor el estado de las máquinas.

Habilite el análisis de aplicaciones adaptables en Security Center para las aplicaciones que existen en su entorno.

- [Habilitación del inventario de máquinas virtuales de Azure](../automation/automation-tutorial-installed-software.md)

- [Uso de los controles de aplicaciones adaptables de Azure Security Center](../security-center/security-center-adaptive-application.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="69-use-only-approved-azure-services"></a>6.9: Uso exclusivo de servicios de Azure aprobados

**Guía**: Use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones del cliente con las siguientes definiciones de directiva integradas:

- Tipos de recursos no permitidos

- Tipos de recursos permitidos

Use Azure Resource Graph para consultar o detectar recursos en las suscripciones. Asegúrese de que todos los recursos de Azure presentes en el entorno estén aprobados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Denegación de un tipo de recurso específico con Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#general)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10: Mantenimiento de un inventario de títulos de software aprobados

**Guía**: use Azure Policy para establecer restricciones sobre el tipo de recursos que se pueden crear en las suscripciones con las definiciones de directiva integradas.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11: Limitación de la capacidad de los usuarios para interactuar con Azure Resource Manager

**Instrucciones**: use el acceso condicional de Azure para limitar la capacidad de los usuarios de interactuar con Azure Resource Manager mediante la configuración de la opción "Bloquear acceso" en la aplicación "Administración de Microsoft Azure".
- [Configuración del acceso condicional para bloquear el acceso a Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="612-limit-users-ability-to-execute-scripts-in-compute-resources"></a>6.12: Limitación de la capacidad de los usuarios para ejecutar scripts en los recursos de proceso

**Guía**: Azure Kubernetes Service (AKS) por sí solo no proporciona una solución de administración de identidades en la que se almacenan contraseñas y cuentas de usuario normales. En su lugar, use Azure Active Directory (Azure AD) como solución de identidad integrada para los clústeres de AKS.

Conceda a los usuarios o grupos acceso a los recursos de Kubernetes de un espacio de nombres o del clúster mediante la integración de Azure AD.

Use el módulo de PowerShell de Azure AD para realizar consultas ad hoc a fin de detectar cuentas que forman parte de grupos administrativos de AKS y usarlas para conciliar el acceso periódicamente. Use la CLI de Azure para operaciones, como "Get access credentials for a managed Kubernetes cluster" ("Obtener credenciales de acceso para un clúster de Kubernetes administrado"). Implemente las recomendaciones de administración de identidad y acceso de Security Center.

- [Administración de AKS con la CLI de Azure](/cli/azure/aks)

- [Descripción de la integración de AKS y Azure AD](concepts-identity.md)

- [Integración de AKS con Azure AD](azure-ad-integration-cli.md)

- [Obtención de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

- [Obtención de los miembros de un rol de directorio en Azure AD con PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

- [Supervisión de identidad y acceso con Azure Security Center](../security-center/security-center-identity-access.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13: Segregación física o lógica de aplicaciones de alto riesgo

**Guía**: Use las características de Azure Kubernetes Service (AKS) a fin de aislar lógicamente los equipos y las cargas de trabajo en el mismo clúster para el número mínimo de privilegios, en el ámbito de los recursos necesarios para cada equipo. 

Implemente el espacio de nombres en Kubernetes para crear un límite de aislamiento lógico. Use alias de Azure Policy en el espacio de nombres "Microsoft.ContainerService" para crear directivas personalizadas a fin de auditar o aplicar la configuración de las instancias de Azure Kubernetes Service (AKS). 

Revise e implemente las características adicionales de Kubernetes y tenga en cuenta los aspectos relativos al aislamiento y la arquitectura multiinquilino para incluir lo siguiente: programación, redes, autenticación o autorización y contenedores. Así mismo, use suscripciones y grupos de administración independientes para los entornos de desarrollo, pruebas y producción. Separe los clústeres de AKS con redes virtuales y subredes que se etiquetan adecuadamente y se protegen con un firewall de aplicaciones web (WAF).

- [Información sobre los procedimientos recomendados para el aislamiento de clústeres en AKS](operator-best-practices-cluster-isolation.md)

- [Creación de suscripciones adicionales de Azure](../cost-management-billing/manage/create-subscription.md)

- [Creación de grupos de administración](../governance/management-groups/create-management-group-portal.md)

- [Descripción de los procedimientos recomendados con la conectividad de red y la seguridad en AKS](operator-best-practices-network.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="secure-configuration"></a>Configuración segura

*Para más información, consulte [Azure Security Benchmark: configuración segura](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1: Establezca configuraciones seguras para todos los recursos de Azure

**Guía**: Use alias de Azure Policy en el espacio de nombres "Microsoft.ContainerService" para crear directivas personalizadas a fin de auditar o aplicar la configuración de las instancias de Azure Kubernetes Service (AKS). Use definiciones de Azure Policy integradas.

Entre los ejemplos de las definiciones de directiva integradas para AKS se incluyen los siguientes:

- Exigir la entrada de HTTPS en el clúster de Kubernetes

- Los intervalos IP autorizados deben definirse en los servicios de Kubernetes

- Se debe usar el control de acceso basado en rol en los servicios de Kubernetes

- Asegurarse de que solo se admiten las imágenes de contenedor permitidas en el clúster de Kubernetes

Exporte una plantilla de la configuración de AKS en notación de objetos JavaScript (JSON) con Azure Resource Manager. Revísela periódicamente para asegurarse de que estas configuraciones cumplan los requisitos de seguridad de su organización. Use las recomendaciones de Azure Security Center como línea base de configuración segura para los recursos de Azure. 

- [Configuración y administración de las directivas de seguridad de pods de AKS](use-pod-security-policies.md)

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="72-establish-secure-operating-system-configurations"></a>7.2: Establezca configuraciones del sistema operativo seguras

**Guía**: Los clústeres de Azure Kubernetes Service (AKS) se implementan en máquinas virtuales del host con un sistema operativo optimizado para la seguridad. El sistema operativo del host cuenta con pasos adicionales de mejora de la seguridad incorporados para reducir el área expuesta de ataque y permitir la implementación de contenedores de manera segura. 

Azure aplica revisiones diarias (lo que incluye revisiones de seguridad) a los hosts de máquina virtual de AKS con alunas revisiones que requieren un reinicio. Los clientes son responsables de programar los reinicios de host de máquina virtual de AKS según sea necesario. 

- [Protección de seguridad para el sistema operativo host del nodo del agente de AKS](security-hardened-vm-host-image.md)

- [Descripción de la mejora de seguridad de los hosts de máquinas virtuales de AKS](security-hardened-vm-host-image.md)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Mantenga configuraciones de recursos de Azure seguras

**Guía**: Proteja su clúster de Azure Kubernetes Service (AKS) mediante directivas de seguridad de pods.  Para mejorar la seguridad del clúster, limite los pods que se pueden programar. 

Los pods que solicitan recursos no permitidos no se pueden ejecutar en el clúster de AKS. 

También puede usar los efectos [deny] y [deploy if not exist] de Azure Policy a fin de aplicar configuraciones seguras para los recursos de Azure relacionados con las implementaciones de AKS (como redes virtuales, subredes, firewalls de Azure, cuentas de almacenamiento, etc.). 

Cree definiciones de Azure Policy personalizadas con alias de los siguientes espacios de nombres: 

- Microsoft.ContainerService

- Microsoft.Network

Información adicional disponible en los vínculos indicados.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Descripción de los efectos de Azure Policy](../governance/policy/concepts/effects.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4: Mantenga configuraciones del sistema operativo seguras

**Guía**: Los clústeres de Azure Kubernetes Service (AKS) se implementan en máquinas virtuales del host con un sistema operativo optimizado para la seguridad. El sistema operativo del host cuenta con pasos adicionales de mejora de la seguridad incorporados para reducir el área expuesta de ataque y permitir la implementación de contenedores de manera segura. 

Consulte la lista de controles de Center for Internet Security (CIS) que están integrados en el sistema operativo del host.  

- [Protección de seguridad para el sistema operativo host del nodo del agente de AKS](security-hardened-vm-host-image.md)

- [Descripción de la configuración de estado de los clústeres de AKS](https://docs.microsoft.com/azure/aks/concepts-clusters-workloads#control-plane)

- [Descripción de la mejora de seguridad de los hosts de máquinas virtuales de AKS](security-hardened-vm-host-image.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Almacene de forma segura la configuración de los recursos de Azure

**Guía**: Use Azure Repos para almacenar y administrar de forma segura las configuraciones si usa definiciones de Azure Policy personalizadas. Exporte una plantilla de la configuración de Azure Kubernetes Service (AKS) en notación de objetos JavaScript (JSON) con Azure Resource Manager. Revísela periódicamente para asegurarse de que las configuraciones cumplan los requisitos de seguridad de su organización.

Implemente soluciones de terceros, como Terraform, para crear un archivo de configuración que declare los recursos para el clúster de Kubernetes. Puede proteger la implementación de AKS si implementa los procedimientos recomendados de seguridad y almacena la configuración como código en una ubicación segura.

- [Definición de un clúster de Kubernetes](/azure/developer/terraform/create-k8s-cluster-with-tf-and-aks#define-a-kubernetes-cluster)

- [Protección de seguridad para el sistema operativo host del nodo del agente de AKS](security-hardened-vm-host-image.md)

- [Almacenamiento de código en Azure DevOps](/azure/devops/repos/git/gitworkflow)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="76-securely-store-custom-operating-system-images"></a>7.6: Almacene imágenes de sistema operativo personalizadas de forma segura

**Guía**: No es aplicable a Azure Kubernetes Service (AKS). AKS proporciona de forma predeterminada un sistema operativo de host optimizado para la seguridad. Actualmente, no hay ninguna opción para seleccionar otro sistema operativo o uno personalizado.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7: Implementación de herramientas de administración de configuración para recursos de Azure

**Guía**: use Azure Policy para aplicar restricciones en el tipo de recursos que se pueden crear en las suscripciones mediante definiciones de directiva integradas, así como alias de Azure Policy en el espacio de nombres "Microsoft.ContainerService". 

Cree directivas personalizadas para auditar y aplicar configuraciones del sistema. Desarrolle un proceso y una canalización para administrar las excepciones de las directivas.

- [Configuración y administración de Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Uso de alias](https://docs.microsoft.com/azure/governance/policy/concepts/definition-structure#aliases)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8: Implementación de herramientas de administración de la configuración para sistemas operativos

**Guía**: Los clústeres de Azure Kubernetes Service (AKS) se implementan en máquinas virtuales del host con un sistema operativo optimizado para la seguridad. El sistema operativo del host cuenta con pasos adicionales de mejora de la seguridad incorporados para reducir el área expuesta de ataque y permitir la implementación de contenedores de manera segura. 

Consulte la lista de controles de Center for Internet Security (CIS) que están integrados en los hosts de AKS.  

- [Protección de seguridad para el sistema operativo host del nodo del agente de AKS](security-hardened-vm-host-image.md)

- [Descripción de la mejora de seguridad de los hosts de máquinas virtuales de AKS](security-hardened-vm-host-image.md)

- [Descripción de la configuración de estado de los clústeres de AKS](https://docs.microsoft.com/azure/aks/concepts-clusters-workloads#control-plane)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9: Implementación de la supervisión de la configuración automatizada para los recursos de Azure

**Guía**: Use Security Center para realizar exámenes de base de referencia de los recursos relacionados con sus implementaciones de Azure Kubernetes Service (AKS). Entre los recursos de ejemplo se incluyen el propio clúster de AKS, la red virtual en la que se implementó el clúster de AKS, la cuenta de Azure Storage usada para realizar un seguimiento del estado de Terraform o las instancias de Azure Key Vault que se usan para las claves de cifrado de los discos de datos y del sistema operativo del clúster de AKS.

- [Corrección de recomendaciones en Azure Security Center](../security-center/security-center-remediate-recommendations.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10: Implemente la supervisión de configuración automatizada para sistemas operativos

**Guía**: Use las recomendaciones de contenedor de Security Center de la sección "Cálculo y aplicaciones" para realizar exámenes de base de referencia de los clústeres de Azure Kubernetes Service (AKS). 

Obtenga notificaciones en el panel de Security Center cuando se encuentren problemas de configuración o vulnerabilidades. Para ello, es necesario habilitar la agrupación opcional de registros de contenedor, que permite a Security Center examinar la imagen.  

- [Descripción de las recomendaciones sobre contenedores de Azure Security Center](../security-center/container-security.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="711-manage-azure-secrets-securely"></a>7.11: Administre los secretos de Azure de forma segura

**Guía**: Integre Azure Key Vault con un clúster de Azure Kubernetes Service (AKS) mediante un controlador FlexVolume. Use Key Vault para almacenar y rotar periódicamente secretos, como credenciales, claves de cuenta de almacenamiento o certificados. El controlador FlexVolume permite que el clúster de AKS pueda recuperar de forma nativa las credenciales de Key Vault y proporcionárselas de forma segura solo al pod solicitante. Use una identidad administrada de pods para solicitar acceso a Key Vault y recuperar las credenciales que necesita mediante el controlador FlexVolume. Asegúrese de que la eliminación temporal de Key Vault esté habilitada. 

Para limitar la exposición de credenciales, no defina credenciales en el código de la aplicación. 

Evite el uso de credenciales fijas o compartidas. 

- [Conceptos de seguridad de las aplicaciones y los clústeres en Azure Kubernetes Service (AKS)](concepts-security.md)

- [Uso de Key Vault con un clúster de AKS](https://docs.microsoft.com/azure/aks/developer-best-practices-pod-security#limit-credential-exposure)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="712-manage-identities-securely-and-automatically"></a>7.12: Administre las identidades de forma segura y automática

**Guía**: No defina las credenciales en el código de aplicación como procedimiento recomendado de seguridad. Use identidades administradas para recursos de Azure con tal de permitir que un pod se autentique en cualquier servicio de Azure que lo admita, como Azure Key Vault. Se asigna una identidad de Azure al pod para que se autentique en Azure Active Directory (Azure AD) y reciba un token digital que se puede presentar en otros servicios de Azure que comprueban si el pod tiene autorización para acceder al servicio y realizar las acciones necesarias.

Tenga en cuenta que las identidades administradas de pods está pensadas para usarse únicamente con pods de Linux y las imágenes de contenedor. Aprovisione Azure Key Vault para almacenar y recuperar credenciales y claves digitales. Las claves como las que se usan para cifrar los discos del sistema operativo o los datos del clúster de AKS se pueden almacenar en Azure Key Vault.

Las entidades de servicio también se pueden usar en clústeres de AKS. Sin embargo, es posible que los clústeres que usan entidades de servicio terminen alcanzando un estado en el que es necesario renovar la entidad de servicio para mantener el clúster en funcionamiento. La administración de entidades de servicio agrega complejidad, por lo que es más fácil usar identidades administradas. Se aplican los mismos requisitos de permisos tanto en las entidades de servicio como en las identidades administradas.

- [Descripción de las identidades administradas y Key Vault con Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/developer-best-practices-pod-security#limit-credential-exposure)

- [Azure AD Pod Identity](https://github.com/Azure/aad-pod-identity)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13: Elimine la exposición de credenciales no intencionada

**Instrucciones**: Implemente el escáner de credenciales para identificar las credenciales en el código. El escáner de credenciales también fomenta el traslado de credenciales detectadas a ubicaciones más seguras, como Azure Key Vault, con recomendaciones.

Para limitar la exposición de credenciales, no defina credenciales en el código de la aplicación y evite el uso de credenciales compartidas. Azure Key Vault se debe usar para almacenar y recuperar claves digitales y credenciales. Use identidades administradas para los recursos de Azure para permitir que el pod solicite acceso a otros recursos. 

- [Configuración del escáner de credenciales](https://secdevtools.azurewebsites.net/helpcredscan.html)

- [Procedimientos recomendados sobre la seguridad de los pods para desarrolladores](developer-best-practices-pod-security.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="malware-defense"></a>Defensa contra malware

*Para más información, consulte [Azure Security Benchmark: defensa contra malware](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-antimalware-software"></a>8.1: Uso de software antimalware administrado centralmente

**Guía**: AKS administra el ciclo de vida y las operaciones de los nodos de agente en su nombre, por lo que no admite la modificación de los recursos de IaaS asociados a los nodos de agente. Sin embargo, en el caso de los nodos de Linux, puede usar conjuntos de demonio para instalar software personalizado como una solución antimalware.

- [Guía de referencia de las alertas de seguridad](../security-center/alerts-reference.md)

- [Alertas para contenedores: clústeres de Azure Kubernetes Service](https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-akscluster)

- [Conjuntos de demonios y responsabilidad compartida de AKS](https://docs.microsoft.com/azure/aks/support-policies#shared-responsibility)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2: Examine previamente los archivos que se van a cargar en recursos de Azure que no son de proceso

**Guía**: Examine previamente los archivos que se cargan en los recursos de AKS. Use la detección de amenazas de Security Center para los servicios de datos a fin de detectar malware cargado en cuentas de almacenamiento si usa una cuenta de Azure Storage como almacén de datos o para realizar un seguimiento del estado de Terraform del clúster de AKS. 

- [Descripción de la detección de amenazas de Azure Security Center para servicios de datos](../security-center/azure-defender.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="83-ensure-antimalware-software-and-signatures-are-updated"></a>8.3: Garantía de la actualización del software y las firmas antimalware

**Guía**: AKS administra el ciclo de vida y las operaciones de los nodos de agente en su nombre, por lo que no admite la modificación de los recursos de IaaS asociados a los nodos de agente. Sin embargo, en el caso de los nodos de Linux, puede usar conjuntos de demonio para instalar software personalizado como una solución antimalware.

- [Guía de referencia de las alertas de seguridad](../security-center/alerts-reference.md)

- [Alertas para contenedores: clústeres de Azure Kubernetes Service](https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-akscluster)

- [Conjuntos de demonios y responsabilidad compartida de AKS](https://docs.microsoft.com/azure/aks/support-policies#shared-responsibility)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

## <a name="data-recovery"></a>Recuperación de datos

*Para más información, consulte [Azure Security Benchmark: recuperación de datos](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1: Garantía de copias de seguridad automáticas periódicas

**Guía**: Realice una copia de seguridad de los datos con una herramienta adecuada para su tipo de almacenamiento, como Velero, que puede realizar copias de seguridad de volúmenes persistentes junto con recursos de clúster y configuraciones adicionales. Verifique de forma periódica la integridad y seguridad de las copias de seguridad. 

Quite el estado de las aplicaciones antes de realizar la copia de seguridad. Cuando no sea posible, realice una copia de seguridad de los datos de los volúmenes persistentes y pruebe regularmente las operaciones de restauración para verificar la integridad de datos y los procesos necesarios.

- [Procedimientos recomendados para el almacenamiento y las copias de seguridad en AKS](operator-best-practices-storage.md)

- [Procedimientos recomendados para la continuidad empresarial y recuperación ante desastres en AKS](operator-best-practices-multi-region.md)

- [Descripción de Azure Site Recovery](../site-recovery/site-recovery-overview.md)

- [Configuración de Velero en Azure](https://github.com/vmware-tanzu/velero-plugin-for-microsoft-azure/blob/master/README.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2: Copias de seguridad completas del sistema y copia de seguridad de las claves administradas por el cliente

**Guía**: Realice una copia de seguridad de los datos con una herramienta adecuada para su tipo de almacenamiento, como Velero, que puede realizar copias de seguridad de volúmenes persistentes junto con recursos de clúster y configuraciones adicionales. 

Realice copias de seguridad automatizadas periódicas de los certificados, las claves, las cuentas de almacenamiento administradas y los secretos de Key Vault con los comandos de PowerShell. 

- [Realización de copias de seguridad de certificados de Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate)

- [Procedimiento para realizar copias de seguridad de claves de Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

- [Realización de copias de seguridad de cuentas de almacenamiento administradas de Key Vault](/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount)

- [Realización de copias de seguridad de secretos de Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret)

- [Habilitación de Azure Backup](/azure/backup/)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3: Validación de todas las copias de seguridad, incluidas las claves administradas por el cliente

**Guía**: Realice periódicamente la restauración de datos de contenido en la copia de seguridad de Velero. Si es necesario, pruebe la restauración en una red virtual o suscripción aislada.

Realice restauraciones de datos periódicas de los certificados, las claves, las cuentas de almacenamiento administradas y los secretos de Key Vault con los comandos de PowerShell.

- [Restauración de certificados de Key Vault](/powershell/module/az.keyvault/restore-azkeyvaultcertificate)

- [Restauración de claves de Key Vault](/powershell/module/az.keyvault/restore-azkeyvaultkey)

- [Restauración de cuentas de Storage de Key Vault](/powershell/module/az.keyvault/backup-azkeyvaultmanagedstorageaccount)

- [Restauración de secretos de Key Vault](/powershell/module/az.keyvault/restore-azkeyvaultsecret)

- [Recuperación de archivos desde una copia de seguridad de máquina virtual de Azure](/azure/backup/backup-azure-restore-files-from-vm)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4: Garantía de la protección de las copias de seguridad y las claves administradas por el cliente

**Guía**: Realice una copia de seguridad de los datos con una herramienta adecuada para su tipo de almacenamiento, como Velero, que puede realizar copias de seguridad de volúmenes persistentes junto con recursos de clúster y configuraciones adicionales. 

Habilite la eliminación temporal en Key Vault para proteger las claves contra la eliminación accidental o malintencionada si Azure Key Vault se usa con las implementaciones de Azure Kubernetes Service (AKS).

- [Descripción de Azure Storage Service Encryption](../storage/common/storage-service-encryption.md)

- [Habilitación de la eliminación temporal en Key Vault](https://docs.microsoft.com/azure/storage/blobs/soft-delete-blob-overview?tabs=azure-portal)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

## <a name="incident-response"></a>Respuesta a los incidentes

*Para más información, consulte [Azure Security Benchmark: respuesta ante incidentes](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1: Creación de una guía de respuesta ante incidentes

**Instrucciones**: Cree una guía de respuesta a incidentes para su organización. Asegúrese de que haya planes de respuesta a incidentes escritos que definan todos los roles del personal, así como las fases de administración y gestión de los incidentes, desde la detección hasta la revisión posterior a la incidencia.

- [Configuración de las automatizaciones de flujos de trabajo en Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Guía para crear su propio proceso de respuesta a incidentes de seguridad](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomía de un incidente del Centro de respuestas de seguridad de Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Guía del NIST para controlar incidentes de seguridad en equipos](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2: Creación de un procedimiento de priorización y puntuación de incidentes

**Guía**: Priorice aquellas alertas que se deben investigar en primer lugar mediante la gravedad que asigna Security Center a las alertas. La gravedad se basa en la confianza que tiene Security Center en la búsqueda o en el análisis utilizados para emitir la alerta, así como en el nivel de confianza de que ha habido un intento malintencionado detrás de la actividad que ha provocado la alerta.
Marque las suscripciones de forma clara (por ejemplo, producción o no producción) y cree un sistema de nomenclatura para identificar y clasificar claramente los recursos de Azure.

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="103-test-security-response-procedures"></a>10.3: Prueba de los procedimientos de respuesta de seguridad

**Guía**: realice ejercicios para probar las capacidades de respuesta a los incidentes de los sistemas con regularidad. Identifique puntos débiles y brechas y revise el plan de respuesta a los incidentes según sea necesario.

- [Guía para probar, entrenar y ejecutar programas para planes y funcionalidades de TI](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4: Provisión de detalles de contacto de incidentes de seguridad y configuración de notificaciones de alerta para incidentes de seguridad

**Guía**: La información de contacto del incidente de seguridad la utilizará Microsoft para ponerse en contacto con usted si Microsoft Security Response Center (MSRC) detecta que un tercero no autorizado o ilegal ha accedido a los datos del cliente. 

Revise los incidentes después del hecho para asegurarse de que se resuelven los problemas.

- [Establecimiento del contacto de seguridad de Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Responsabilidad**: Customer

**Supervisión de Azure Security Center**: ninguna

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5: Incorporación de alertas de seguridad en el sistema de respuesta a incidentes

**Guía**: exporte alertas y recomendaciones de Security Center mediante su característica de exportación continua. La exportación continua le permite exportar alertas y recomendaciones de forma manual o continua. 

Elija el conector de datos de Security Center para transmitir las alertas a Azure Sentinel según sea necesario y de acuerdo con los requisitos organizativos de la empresa.

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

**Guía**: Siga las reglas de compromiso de Microsoft para asegurarse de que las pruebas de penetración no infrinjan las directivas de Microsoft. Más información sobre la estrategia y puesta en marcha de un equipo rojo de Microsoft, y las pruebas de penetración en sitios activos de la infraestructura en la nube, las aplicaciones y los servicios administrados por Microsoft en los vínculos a los que se hace referencia.

- [Reglas de interacción de las pruebas de penetración](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Equipo rojo de Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Responsabilidad**: Compartido

**Supervisión de Azure Security Center**: ninguna

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [Información general sobre Azure Security Benchmark V2](/azure/security/benchmarks/overview).
- Obtenga más información sobre las [líneas de base de seguridad de Azure](/azure/security/benchmarks/security-baselines-overview).
