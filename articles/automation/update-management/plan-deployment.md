---
title: Planeamiento de la implementación de Update Management de Azure Automation
description: En este artículo se describen las consideraciones y decisiones que se deben tomar para preparar la implementación de Update Management de Azure Automation.
services: automation
ms.subservice: update-management
ms.date: 06/07/2021
ms.topic: conceptual
ms.openlocfilehash: 5b2303b17a525a6ae394996780fe3ebb89388f44
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111855158"
---
# <a name="plan-your-update-management-deployment"></a>Planeamiento de la implementación de Update Management

## <a name="step-1---automation-account"></a>Paso 1: Cuenta de Azure Automation

Update Management es una característica de Azure Automation y, por tanto, requiere una cuenta de Automation. Puede usar una cuenta de Automation existente en su suscripción o crear una nueva cuenta dedicada solo para Update Management y para ninguna otra función de Automation.

## <a name="step-2---azure-monitor-logs"></a>Paso 2: Registros de Azure Monitor

Update Management depende de un área de trabajo de Log Analytics en Azure Monitor para almacenar los datos de registro de estado de evaluación y actualización recopilados de las máquinas administradas. La integración con Log Analytics también permite el análisis detallado y las alertas en Azure Monitor. Puede usar un área de trabajo existente en su suscripción, o puede crear una nueva dedicada solo para Update Management.

Si no conoce bien los registros de Azure Monitor y el área de trabajo de Log Analytics, consulte la guía de implementación [Diseño de un área de trabajo de Log Analytics](../../azure-monitor/logs/design-logs-deployment.md). 

## <a name="step-3---supported-operating-systems"></a>Paso 3: Sistemas operativos admitidos

Update Management admite versiones específicas de los sistemas operativos Windows Server y Linux. Antes de habilitar Update Management, confirme que las máquinas de destino cumplen los [requisitos de sistema operativo](operating-system-requirements.md). 

## <a name="step-4---log-analytics-agent"></a>Paso 4: Agente de Log Analytics

Se necesita el [agente de Log Analytics](../../azure-monitor/agents/log-analytics-agent.md) para Windows y Linux para admitir Update Management. El agente se usa tanto para la recopilación de datos como para el rol de Hybrid Runbook Worker del sistema de Automation que admite los runbooks de Update Management que se usan para administrar la evaluación e implementación de actualizaciones en la máquina. 

En las máquinas virtuales de Azure, si el agente de Log Analytics aún no está instalado, se instala automáticamente al habilitar Update Management para la máquina virtual, mediante la extensión de máquina virtual de Log Analytics para [Windows](../../virtual-machines/extensions/oms-windows.md) o [Linux](../../virtual-machines/extensions/oms-linux.md). El agente está configurado para informar al área de trabajo de Log Analytics vinculada a la cuenta de Automation en la que Update Management está habilitada.

Los servidores o máquinas virtuales que no son de Azure deben tener instalado el agente de Log Analytics para Windows o Linux e informar al área de trabajo vinculada. Se recomienda instalar el agente de Log Analytics para Windows o Linux. Para ello, primero debe conectar la máquina a los [servidores habilitados para Azure Arc](../../azure-arc/servers/overview.md) y, a continuación, use Azure Policy para asignar el [agente de Log Analytics a la directiva integrada de las máquinas de Azure Arc de Linux o de Windows](../../governance/policy/samples/built-in-policies.md#monitoring). Como alternativa, si planea supervisar las máquinas con [VM Insights](../../azure-monitor/vm/vminsights-overview.md), en su lugar, use la iniciativa [Habilitar Azure Monitor para VM](../../governance/policy/samples/built-in-initiatives.md#monitoring).

Si va a habilitar una máquina que Operations Manager administre, no es necesario un nuevo agente. La información del área de trabajo se agrega a la configuración de los agentes cuando se conecta el grupo de administración al área de trabajo de Log Analytics.

No se permite tener registrada una máquina para Update Management en más de un área de trabajo de Log Analytics (también conocido como hospedaje múltiple).

## <a name="step-5---network-planning"></a><a name="ports"></a> Paso 5: Planeamiento de red

Para preparar la red para admitir Update Management, es posible que tenga que configurar algunos componentes de la infraestructura. Por ejemplo, abra los puertos de firewall para pasar las comunicaciones usadas por Update Management y Azure Monitor.

Consulte [Configuración de red de Azure Automation](../automation-network-configuration.md) para más información sobre los puertos, las direcciones URL y otros detalles de red necesarios para Update Management, incluido el rol Hybrid Runbook Worker. Para conectarse al servicio Automation desde las máquinas virtuales de Azure de forma segura y privada, revise [Uso de Azure Private Link](../how-to/private-link-security.md). 

En el caso de las máquinas Windows, también debe permitir el tráfico a los puntos de conexión que el agente de Windows Update necesita. Puede encontrar una lista actualizada de los puntos de conexión necesarios en [Problemas relacionados con HTTP/Proxy](/windows/deployment/update/windows-update-troubleshooting#issues-related-to-httpproxy). Si tiene una implementación local de [Windows Server Update Services](/windows-server/administration/windows-server-update-services/plan/plan-your-wsus-deployment) (WSUS), también debe permitir el tráfico al servidor especificado en su [clave de WSUS](/windows/deployment/update/waas-wu-settings#configuring-automatic-updates-by-editing-the-registry).

En el caso de máquinas de Red Hat Linux, consulte las [direcciones IP de los servidores de entrega de contenido de RHUI](../../virtual-machines/workloads/redhat/redhat-rhui.md#the-ips-for-the-rhui-content-delivery-servers) de los puntos de conexión necesarios. Para otras distribuciones de Linux, consulte la documentación del proveedor.

Si las directivas de seguridad de TI no permiten que las máquinas de la red se conecten a Internet, puede configurar una [puerta de enlace de Log Analytics](../../azure-monitor/agents/gateway.md) y, luego, configurar la máquina para conectarse a Azure Monitor y a Azure Automation mediante la puerta de enlace.

## <a name="step-6---permissions"></a>Paso 6: Permisos

Para crear y administrar implementaciones de actualizaciones, necesita permisos concretos. Para más información sobre estos permisos, consulte [Acceso basado en rol: Update Management](../automation-role-based-access-control.md#update-management-permissions).

## <a name="step-7---windows-update-client"></a>Paso 7: Cliente de Windows Update

Azure Automation Update Management se basa en el cliente de Windows Update para descargar e instalar las actualizaciones de Windows. Hay una configuración de directiva de grupo específica que Windows Update Agent (WUA) usa en las máquinas para conectarse a Windows Server Update Services (WSUS) o Microsoft Update. Esta configuración de directiva de grupo también se utiliza para analizar correctamente el cumplimiento de las actualizaciones de software, y para implementar automáticamente las actualizaciones de software. Para revisar nuestras recomendaciones, consulte [Configuración de Windows Update para Update Management](configure-wuagent.md).

## <a name="step-8---linux-repository"></a>Paso 8: Repositorio de Linux

Las máquinas virtuales que se crearon a partir de imágenes a petición de Red Hat Enterprise Linux (RHEL), disponibles en Azure Marketplace, están registradas para acceder a la instancia de Red Hat Update Infrastructure (RHUI) implementada en Azure. Cualquier otra distribución de Linux se debe actualizar desde el repositorio de archivos en línea de la distribución con los métodos que esta admite.

Para clasificar las actualizaciones de la versión 6 de Red Hat Enterprise, debe instalar el complemento de seguridad de yum. En Red Hat Enterprise Linux 7, el complemento ya forma parte de yum y no es necesario instalar nada. Para más información, consulte el siguiente [artículo de conocimientos](https://access.redhat.com/solutions/10021) de Red Hat.

## <a name="step-9---plan-deployment-targets"></a>Paso 9: Planeamiento de los destinos de implementación

Update Management permite dirigir las actualizaciones a un grupo dinámico que representa máquinas de Azure o no de Azure, para asegurarse de que cada máquina obtenga siempre las actualizaciones correctas en el momento más conveniente. Un grupo dinámico se resuelve en tiempo de implementación y se basa en los criterios siguientes:

* Subscription
* Grupos de recursos
* Ubicaciones
* Etiquetas 

En el caso de las máquinas que no son de Azure, un grupo dinámico usa búsquedas guardadas, también llamadas [grupos de equipos](../../azure-monitor/logs/computer-groups.md). Las implementaciones de actualizaciones con ámbito en un grupo de máquinas solo son visibles desde la cuenta de Automation, en la opción **Programaciones de implementación** de Update Management, no desde una máquina virtual de Azure específica.

Las actualizaciones se pueden administrar también solo para una máquina virtual de Azure seleccionada. Las implementaciones de actualizaciones con ámbito en la máquina específica son visibles tanto desde la máquina como desde la cuenta de Automation, en **Programaciones de implementación** de Update Management. 

## <a name="next-steps"></a>Pasos siguientes

Habilite Update Management y seleccione las máquinas que se administrarán con uno de los métodos siguientes:

- Use una [plantilla de Azure Resource Manager](enable-from-template.md) para implementar Update Management en una cuenta de Automation nueva o existente, y el área de trabajo de Log Analytics de Azure Monitor en la suscripción. No configura el ámbito de las máquinas que deben administrarse; esto se realiza como un paso independiente después de usar la plantilla.

- Desde la [cuenta de Automation](enable-from-automation-account.md) para una o varias máquinas de Azure o que no sean de Azure, incluidos los servidores habilitados para Arc.

- Use el [runbook](enable-from-runbook.md) **Enable-AutomationSolution** para automatizar la incorporación de VM de Azure.

- Para una [VM de Azure concreta](enable-from-vm.md), desde la página **Máquinas virtuales** en Azure Portal. Este escenario está disponible para las VM Linux y Windows.

- Para [varias máquinas virtuales de Azure](enable-from-portal.md), selecciónelas en la página **Máquinas virtuales** de Azure Portal.