---
title: Introducción a Hybrid Runbook Worker de Azure Automation
description: En este artículo se ofrece información general de Hybrid Runbook Worker, que puede usar para ejecutar runbooks en máquinas de su centro de datos local o proveedor de nube.
services: automation
ms.subservice: process-automation
ms.date: 01/22/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2bb178302d399805eb84b233060d5717e2dba8b3
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107830563"
---
# <a name="hybrid-runbook-worker-overview"></a>Introducción a Hybrid Runbook Worker

Es posible que los runbooks de Azure Automation no tengan acceso a los recursos de otras nubes o del entorno local, porque se ejecutan en plataforma de nube de Azure. La característica Hybrid Runbook Worker de Azure Automation permite ejecutar runbooks directamente en la máquina que hospeda el rol y en los recursos del entorno para administrar dichos recursos locales. Los runbooks se almacenan y administran en Azure Automation y después se entregan a una o más máquinas asignadas.

## <a name="runbook-worker-types"></a>Tipos de Runbook Worker

Hay dos tipos de instancias de Runbook Worker: sistema y usuario. En la siguiente tabla se describen las diferencias entre ellas.

|Tipo | Descripción |
|-----|-------------|
|**Sistema** |Admite un conjunto de runbooks ocultos que usa la característica Update Management y están diseñados para instalar actualizaciones especificadas por el usuario en máquinas Windows y Linux.<br> Este tipo de Hybrid Runbook Worker no es miembro de un grupo de instancias de Hybrid Runbook Worker y, por lo tanto, no ejecuta runbooks que tienen como destino un grupo de Runbook Worker. |
|**User** |Admite runbooks definidos por el usuario que se van a ejecutar directamente en la máquina Windows y Linux y son miembros de uno o más grupos de Runbook Worker. |

Una instancia de Hybrid Runbook Worker puede ejecutarse en el sistema operativo Windows o Linux, y este rol depende de que el [agente de Log Analytics](../azure-monitor/agents/log-analytics-agent.md) envíe informes a un [área de trabajo de Log Analytics](../azure-monitor/logs/design-logs-deployment.md) de Azure Monitor. El área de trabajo no sirve solo para supervisar el sistema operativo compatible de la máquina, sino también para descargar los componentes necesarios para instalar Hybrid Runbook Worker.

Si habilita [Update Management](./update-management/overview.md) de Azure Automation, las máquinas conectadas al área de trabajo de Log Analytics se configurarán automáticamente como una instancia de Hybrid Runbook Worker del sistema. Para configurarla como una instancia de Hybrid Runbook Worker de Windows del usuario, consulte [Implementación de Hybrid Runbook Worker en Windows](automation-windows-hrw-install.md) e [Implementación de Hybrid Runbook Worker en Linux](automation-linux-hrw-install.md).

## <a name="how-does-it-work"></a>¿Cómo funciona?

![Introducción a Hybrid Runbook Worker](media/automation-hybrid-runbook-worker/automation.png)

Cada Hybrid Runbook Worker de usuario es miembro de un grupo de instancias de Hybrid Runbook Worker que especifica cuando instala el trabajo. Un grupo puede incluir solo un trabajo, pero puede incluir varios en un grupo para contar con alta disponibilidad. Cada máquina puede hospedar una instancia de Hybrid Runbook Worker que envía informes a una cuenta de Automation; no se puede registrar a la instancia de Hybrid Worker en varias cuentas de Automation. Una instancia de Hybrid Worker solo puede escuchar trabajos de una única cuenta de Automation. Las máquinas que hospedan la instancia de Hybrid Runbook Worker del sistema administrada por Update Management se pueden agregar a un grupo de Hybrid Runbook Worker. Sin embargo, debe usar la misma cuenta de Automation para Update Management y para la pertenencia al grupo de Hybrid Runbook Worker.

Cuando se inicia un runbook en un Hybrid Runbook Worker de usuario, se especifica el grupo en el que se ejecuta. Cada rol de trabajo del grupo sondea Azure Automation para ver si hay trabajos disponibles. Si un trabajo está disponible, el primer rol trabajo en obtener el trabajo lo toma. El tiempo de procesamiento de la cola de trabajos depende de la carga y del perfil de hardware de Hybrid Worker. No se puede especificar un trabajo determinado. Hybrid Worker usa un mecanismo de sondeo (cada 30 segundos) y respeta un orden de servir primero al primero en llegar. En función del momento en que se insertó un trabajo, la instancia de Hybrid Worker que haga ping al servicio Automation toma el trabajo. Normalmente, una sola instancia de Hybrid Worker puede seleccionar cuatro trabajos por ping (es decir, cada 30 segundos). Si la velocidad de inserción de trabajos es superior a cuatro por cada 30 segundos, es muy probable que otra instancia de Hybrid Worker del grupo de Hybrid Runbook Worker realice el trabajo.

Una instancia de Hybrid Runbook Worker no tiene muchos de los [límites](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits) del [espacio aislado de Azure](automation-runbook-execution.md#runbook-execution-environment) en el espacio en disco, la memoria o los sockets de red. Los límites de una instancia de Hybrid Worker solo están relacionados con los propios recursos del trabajo y no están limitados por el límite de tiempo de [distribución equilibrada](automation-runbook-execution.md#fair-share) que tienen los espacios aislados de Azure.

Para controlar la distribución de runbooks en las instancias de Hybrid Runbook Worker y cuándo o cómo se desencadenan los trabajos, puede registrar la instancia de Hybrid Worker en distintos grupos de Hybrid Runbook Worker de la cuenta de Automation. Establezca como destino los trabajos en el grupo o grupos específicos para admitir la organización de ejecuciones.

## <a name="hybrid-runbook-worker-installation"></a>Instalación de una instancia de Hybrid Runbook Worker

El proceso para instalar una instancia de Hybrid Runbook Worker de usuario depende del sistema operativo. En la tabla siguiente, se definen los tipos de implementación.

|Sistema operativo  |Tipos de implementación  |
|---------|---------|
|Windows     | [Automatizado](automation-windows-hrw-install.md#automated-deployment)<br>[Manual](automation-windows-hrw-install.md#manual-deployment)        |
|Linux     | [Manual](automation-linux-hrw-install.md#install-a-linux-hybrid-runbook-worker)        |

El método de instalación recomendado para una máquina Windows consiste en usar un runbook de Azure Automation para automatizar por completo el proceso de configuración. Si no es posible, puede seguir un procedimiento paso a paso para instalar y configurar el rol manualmente. En el caso de las máquinas de Linux, debe ejecutar un script de Python para instalar el agente en la máquina.

## <a name="network-planning"></a><a name="network-planning"></a>Planeamiento de red

Consulte [Configuración de red de Azure Automation](automation-network-configuration.md#network-planning-for-hybrid-runbook-worker) para más información sobre los puertos, las direcciones URL y otros detalles de red necesarios para Hybrid Runbook Worker.

### <a name="proxy-server-use"></a>Uso del servidor proxy

Si se usa un servidor proxy para la comunicación entre Azure Automation y las máquinas que ejecutan el agente de Log Analytics, asegúrese de que sea posible acceder a los recursos adecuados. El tiempo de expiración para las solicitudes de Hybrid Runbook Worker y los servicios de Automation es de 30 segundos. Después de tres intentos, se produce un error en una solicitud.

### <a name="firewall-use"></a>Uso del firewall

Si usa un firewall para restringir el acceso a Internet, tendrá que configurarlo para que permita el acceso. Si usa la puerta de enlace de Log Analytics como proxy, asegúrese de que está configurada para instancias de Hybrid Runbook Worker. Consulte [Configuración de la pueta de enlace de Log Analytics para Hybrid Runbook Worker de Automation](../azure-monitor/agents/gateway.md).

### <a name="service-tags"></a>Etiquetas de servicio

Azure Automation admite etiquetas de servicio de red virtual de Azure, empezando por la etiqueta de servicio [GuestAndHybridManagement](../virtual-network/service-tags-overview.md). Puede usar etiquetas de servicio para definir controles de acceso a la red en [grupos de seguridad de red](../virtual-network/network-security-groups-overview.md#security-rules) o [Azure Firewall](../firewall/service-tags.md). Las etiquetas de servicio se pueden usar en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio **GuestAndHybridManagement** en el campo de origen o destino apropiado de una regla, puede permitir o denegar el tráfico para el servicio Automation. Esta etiqueta de servicio no permite un control más pormenorizado mediante la restricción de los intervalos de IP a una región específica.

La etiqueta de servicio de Azure Automation solo proporciona direcciones IP que se usan para los siguientes escenarios:

* Desencadenar webhooks desde dentro de la red virtual
* Permitir que los agentes de Hybrid Runbook Worker o State Configuration de la red virtual se comuniquen con el servicio Automation

>[!NOTE]
>La etiqueta de servicio **GuestAndHybridManagement** no admite actualmente la ejecución de trabajos de runbook en un espacio aislado de Azure, solo directamente en una instancia de Hybrid Runbook Worker.

## <a name="support-for-impact-level-5-il5"></a>Compatibilidad con el nivel 5 de impacto (IL5)

Hybrid Runbook Worker de Azure Automation se puede usar en Azure Government para admitir cargas de trabajo de nivel 5 de impacto en cualquiera de las dos configuraciones siguientes:

* [Máquina virtual aislada](../azure-government/documentation-government-impact-level-5.md#isolated-virtual-machines). Cuando se implementa, consume todo el host físico de esa máquina, lo que proporciona el nivel necesario de aislamiento para admitir las cargas de trabajo IL5.

* [Azure Dedicated Host](../azure-government/documentation-government-impact-level-5.md#azure-dedicated-host), proporciona servidores físicos capaces de hospedar una o varias máquinas virtuales, dedicados a una suscripción de Azure.

>[!NOTE]
>El aislamiento de proceso mediante el rol Hybrid Runbook Worker está disponible para las nubes de Azure Commercial y US Government.

### <a name="update-management-addresses-for-hybrid-runbook-worker"></a>Direcciones de Update Management para Hybrid Runbook Worker

Además de las direcciones y puertos estándar necesarios para Hybrid Runbook Worker, Update Management tiene otros requisitos de configuración de red que se describen en la sección [Planeamiento de red](./update-management/overview.md#ports).

## <a name="azure-automation-state-configuration-on-a-hybrid-runbook-worker"></a>State Configuration de Azure Automation en una instancia de Hybrid Runbook Worker

Puede ejecutar [State Configuration de Azure Automation](automation-dsc-overview.md) en una instancia de Hybrid Runbook Worker. Para administrar la configuración de los servidores que admiten la instancia de Hybrid Runbook Worker, debe agregar dichos servidores como nodos de DSC. Consulte el artículo [Incorporación de máquinas para su administración mediante Azure Automation State Configuration](automation-dsc-onboarding.md).

## <a name="runbook-worker-limits"></a>Límites de Runbook Worker

El número máximo de grupos de Hybrid Worker por cada cuenta de Automation es 4000, y este límite se aplica a las instancias de Hybrid Worker del sistema y de usuarios. Si tiene que administrar más de 4000 máquinas, se recomienda crear otra cuenta de Automation.

## <a name="runbooks-on-a-hybrid-runbook-worker"></a>Runbooks en Hybrid Runbook Worker

Podría tener runbooks que administren recursos en la máquina local o se ejecuten con recursos en el entorno local en el que se implementa una instancia de Hybrid Runbook Worker de usuario. En este caso, puede elegir ejecutar los runbooks en la instancia de Hybrid Worker en lugar de en una cuenta de Automation. La estructura de los runbooks que se ejecutan en una instancia de Hybrid Runbook Worker es idéntica a la de los que ejecuta en la cuenta de Automation. Consulte [Ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md).

### <a name="hybrid-runbook-worker-jobs"></a>Trabajos de Hybrid Runbook Worker

Los trabajos de Hybrid Runbook Worker se ejecutan en la cuenta del **sistema** local en Windows o en la [cuenta nxautomation](automation-runbook-execution.md#log-analytics-agent-for-linux) en Linux. Azure Automation controla los trabajos en Hybrid Runbook Worker de forma diferente a los trabajos que se ejecutan en espacios aislados de Azure. Consulte [Dónde ejecutar los runbooks](automation-runbook-execution.md#runbook-execution-environment).

Si la máquina host de Hybrid Runbook Worker se reinicia, cualquier trabajo de runbook en ejecución se reinicia desde el principio o desde el último punto de comprobación para runbooks de Flujo de trabajo de PowerShell. Si un trabajo de runbook se reinicia más de tres veces, se suspende.

### <a name="runbook-permissions-for-a-hybrid-runbook-worker"></a>Permisos del runbook para una instancia de Hybrid Runbook Worker

Puesto que acceden a recursos que no son de Azure, los runbooks que se ejecutan en una instancia de Hybrid Runbook Worker de usuario no pueden usar el mecanismo de autenticación que usan normalmente los runbooks que se autentican en los recursos de Azure. Un runbook proporciona su propia autenticación a los recursos locales o configura la autenticación mediante [identidades administradas para los recursos de Azure](../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-arm.md#grant-your-vm-access-to-a-resource-group-in-resource-manager). También puede especificar una cuenta de ejecución para proporcionar un contexto de usuario para todos los runbooks.

## <a name="view-system-hybrid-runbook-workers"></a>Visualización de instancias de Hybrid Runbook Worker del sistema

Una vez habilitada la característica Update Management en máquinas Windows o Linux, puede inventariar la lista de grupos de Hybrid Runbook Worker del sistema en Azure Portal. Puede ver hasta 2000 trabajos en el portal si selecciona la pestaña **System hybrid workers group** (Grupos de trabajos híbridos del sistema) de la opción **Hybrid workers group** (Grupo de trabajos híbridos) en el panel izquierdo de la cuenta de Automation seleccionada.

:::image type="content" source="./media/automation-hybrid-runbook-worker/system-hybrid-workers-page.png" alt-text="Página de grupos de trabajos híbridos del sistema de la cuenta de Automation" border="false" lightbox="./media/automation-hybrid-runbook-worker/system-hybrid-workers-page.png":::

Si tiene más de 2000 trabajos híbridos, puede ejecutar el siguiente script de PowerShell para obtener una lista de todos ellos:

```powershell
"Get-AzSubscription -SubscriptionName "<subscriptionName>" | Set-AzContext
$workersList = (Get-AzAutomationHybridWorkerGroup -ResourceGroupName "<resourceGroupName>" -AutomationAccountName "<automationAccountName>").Runbookworker
$workersList | export-csv -Path "<Path>\output.csv" -NoClobber -NoTypeInformation"
```

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md) para más información sobre cómo configurar los runbooks para automatizar los procesos del centro de datos local o de otros entornos de nube.

* Para aprender a solucionar problemas con las instancias de Hybrid Runbook Worker, consulte [Solución de incidencias de Hybrid Runbook Worker](troubleshoot/hybrid-runbook-worker.md#general).
