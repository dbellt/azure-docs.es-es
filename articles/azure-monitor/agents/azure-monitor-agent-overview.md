---
title: Información general del agente de Azure Monitor
description: Información general del agente de Azure Monitor (AMA), que recopila datos de supervisión del sistema operativo invitado de máquinas virtuales.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/22/2021
ms.custom: references_regions
ms.openlocfilehash: 7c12fff502d8c4b68470370e0a5eede1dd3866a9
ms.sourcegitcommit: 3941df51ce4fca760797fa4e09216fcfb5d2d8f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2021
ms.locfileid: "114605360"
---
# <a name="azure-monitor-agent-overview"></a>Información general del agente de Azure Monitor
El agente de Azure Monitor (AMA) recopila datos de supervisión del sistema operativo invitado de máquinas virtuales de Azure y los entrega a Azure Monitor. En este artículo se proporciona información general sobre el agente de Azure Monitor, incluido cómo instalarlo y cómo configurar la recopilación de datos.

## <a name="relationship-to-other-agents"></a>Relación con otros agentes
El agente de Azure Monitor reemplaza a los siguientes agentes antiguos utilizados actualmente por Azure Monitor para recopilar datos de invitado de máquinas virtuales ([ver brechas conocidas](/azure/azure-monitor/faq#is-the-new-azure-monitor-agent-at-parity-with-existing-agents)):

- [Agente de Log Analytics](./log-analytics-agent.md): envía datos al área de trabajo de Log Analytics y admite VM Insights y soluciones de supervisión.
- [Extensión de Diagnostics](./diagnostics-extension-overview.md): Envía datos a Azure Storage, a las métricas de Azure Monitor (solo en Windows), a Event Hubs y a Azure Storage.
- [Agente de Telegraf](../essentials/collect-custom-metrics-linux-telegraf.md): Envía datos a las métricas de Azure Monitor (solo Linux).

Además de consolidar esta funcionalidad en un solo agente, el agente de Azure Monitor proporciona las siguientes ventajas frente a los agentes existentes:

- Ámbito de supervisión. Configure de forma centralizada la recopilación para diferentes conjuntos de datos de diferentes conjuntos de VM.  
- Hospedaje múltiple en Linux: Envíe datos de VM Linux a varias áreas de trabajo.
- Filtrado de eventos de Windows: Use consultas de XPATCH para filtrar los eventos de Windows que se recopilan.
- Administración mejorada de las extensiones: El agente de Azure Monitor utiliza un nuevo método de control de la extensibilidad que es más transparente y controlable que los módulos de administración y los complementos de Linux en los agentes de Log Analytics actuales.

### <a name="changes-in-data-collection"></a>Cambios en la recopilación de datos
Los métodos para definir la recopilación de datos de los agentes existentes son distintos entre sí, y cada uno de ellos tiene desafíos que se abordan con el agente de Azure Monitor.

- El agente de Log Analytics obtiene su configuración de un área de trabajo de Log Analytics. Esto es fácil de configurar de forma centralizada, pero es difícil establecer definiciones independientes para diferentes máquinas virtuales. Solo puede enviar datos a un área de trabajo de Log Analytics.

- La extensión de Diagnostics tiene una configuración para cada máquina virtual. Resulta fácil establecer definiciones independientes para diferentes máquinas virtuales, pero difícil de administrar de forma centralizada. Solo puede enviar datos a métricas de Azure Monitor, Azure Event Hubs o Azure Storage. En el caso de los agentes de Linux, se necesita el agente de Telegraf de código abierto para enviar datos a las métricas de Azure Monitor.

El agente de Azure Monitor utiliza las [reglas de recopilación de datos (DCR)](data-collection-rule-overview.md) para configurar los datos que se van a recopilar de cada agente. Las reglas de recopilación de datos permiten la capacidad de administración de la configuración de la recopilación a escala y, al mismo tiempo, habilitan configuraciones únicas con ámbito para subconjuntos de máquinas. Son independientes del área de trabajo e independientes de la máquina virtual, lo que permite definirlas una vez y reutilizarlas en distintas máquinas y entornos. Consulte [Configuración de la recopilación de datos para el agente de Azure Monitor](data-collection-rule-azure-monitor-agent.md).

## <a name="should-i-switch-to-azure-monitor-agent"></a>¿Debo cambiar al agente de Azure Monitor agente?
El agente de Azure Monitor reemplaza los [agentes antiguos de Azure Monitor](agents-overview.md) y puede empezar a realizar la transición de las máquinas virtuales fuera de los agentes actuales al nuevo agente teniendo en cuenta los siguientes factores:

- **Requisitos de entorno.** El agente de Azure Monitor admite [estos sistemas operativos](./agents-overview.md#supported-operating-systems) en la actualidad. Lo más probable es que en este nuevo agente se proporcione compatibilidad con las versiones futuras del sistema operativo, con el entorno y requisitos de red. Debe evaluar si el entorno el agente de Azure Monitor admite el entorno. Si no es así, tendrá que permanecer con el agente actual. Si el agente de Azure Monitor admite el entorno actual, debería considerar la posibilidad de realizar la transición a él.
- **Requisitos de características actuales y nuevas.** El agente de Azure Monitor presenta varias funcionalidades nuevas, como el filtrado, el ámbito y el hospedaje múltiple, pero aún no está a la par con los agentes actuales para otras funcionalidades, como la recopilación de registros personalizados y la integración con todas las soluciones ([consulte soluciones en versión preliminar](/azure/azure-monitor/faq#which-log-analytics-solutions-are-supported-on-the-new-azure-monitor-agent)). La mayoría de las nuevas funcionalidades de Azure Monitor solo estarán disponibles con el agente de Azure Monitor, por lo que, con el tiempo, las funcionalidades adicionales solo estarán disponibles en el nuevo agente. Tenga en cuenta si el agente de Azure Monitor tiene las características que necesita y si hay algunas características de las que puede prescindir temporalmente para obtener otras características importantes en el nuevo agente. Si el agente de Azure Monitor tiene todas las funcionalidades principales que necesita, considere la posibilidad de realizar la transición a él. Si hay características críticas que necesita, continúe con el agente actual hasta que el agente de Azure Monitor alcance la paridad.
- **Tolerancia al reprocesamiento.** Si va a configurar un nuevo entorno con recursos como scripts de implementación y plantillas de incorporación, evalúe el esfuerzo que conlleva. Si va a implicar una cantidad significativa de trabajo, considere la posibilidad de configurar el nuevo entorno con el nuevo agente ya que ahora tiene disponibilidad general. Se publicará una fecha de desuso para los agentes de Log Analytics en agosto de 2021. Los agentes actuales se admitirán durante varios años una vez que se inicie el desuso.

## <a name="supported-resource-types"></a>Tipos de recurso admitidos
Actualmente se admiten máquinas virtuales de Azure, conjuntos de escalado de máquinas virtuales y servidores habilitados para Azure Arc. Azure Kubernetes Service y otros tipos de recursos de proceso no se admiten en este momento.


## <a name="supported-regions"></a>Regiones admitidas
El agente de Azure Monitor está disponible en todas las regiones públicas que admiten Log Analytics. Las regiones y nubes gubernamentales no se admiten en este momento.
## <a name="supported-services-and-features"></a>Servicios y características admitidos
En la tabla siguiente se muestra la compatibilidad actual del agente de Azure Monitor con otros servicios de Azure.

| Servicio de Azure | Compatibilidad actual | Más información |
|:---|:---|:---|
| [Azure Security Center](../../security-center/security-center-introduction.md) | Versión preliminar privada | [Vínculo de registro](https://aka.ms/AMAgent) |
| [Azure Sentinel](../../sentinel/overview.md) | Versión preliminar privada | [Vínculo de registro](https://aka.ms/AMAgent) |


En la tabla siguiente se muestra la compatibilidad actual del agente de Azure Monitor con características de Azure.

| Característica de Azure Monitor | Compatibilidad actual | Más información |
|:---|:---|:---|
| [VM Insights](../vm/vminsights-overview.md) | Versión preliminar privada  | [Vínculo de registro](https://forms.office.com/r/jmyE821tTy) |
| [Estado de invitado de VM Insights](../vm/vminsights-health-overview.md) | Versión preliminar pública | Disponible solo en el nuevo agente |
| [SQL Insights](../insights/sql-insights-overview.md) | Versión preliminar pública | Disponible solo en el nuevo agente |

En la tabla siguiente se muestra la compatibilidad actual del agente de Azure Monitor con soluciones de Azure.

| Solución | Compatibilidad actual | Más información |
|:---|:---|:---|
| [Seguimiento de cambios](../../automation/change-tracking/overview.md) | Se admite como Supervisión de la integridad de los archivos (FIM) en la versión preliminar privada de Azure Security Center.  | [Vínculo de registro](https://aka.ms/AMAgent) |
| [Administración de actualizaciones](../../automation/update-management/overview.md) | Use la versión Update Management v2 (versión preliminar privada) que no requiere un agente. | [Vínculo de registro](https://www.yammer.com/azureadvisors/threads/1064001355087872) |



## <a name="coexistence-with-other-agents"></a>Coexistencia con otros agentes
El agente de Azure Monitor puede coexistir con los agentes existentes para que pueda seguir usando su funcionalidad existente durante la evaluación o la migración. Esto es especialmente importante debido a las limitaciones para admitir las soluciones existentes. Sin embargo, debe tener cuidado al recopilar datos duplicados, ya que esto podría sesgar los resultados de las consultas y generar cargos adicionales por la ingesta y retención de datos. 

Por ejemplo, VM Insights usa el agente de Log Analytics para enviar datos de rendimiento a un área de trabajo de Log Analytics. También puede haber configurado el área de trabajo para recopilar eventos de Windows y eventos de Syslog de los agentes. Si instala el agente de Azure Monitor y crea una regla de recopilación de datos para estos mismos eventos y datos de rendimiento, se generarán datos duplicados.

Por lo tanto, asegúrese de que no recopila los mismos datos de ambos agentes. Si está seguro, compruebe que van a destinos independientes.


## <a name="costs"></a>Costos
El agente de Azure Monitor no cuesta nada, pero puede incurrir en gastos por los datos ingeridos. Consulte [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/) para obtener más información sobre la recopilación y retención de datos de Log Analytics, y para las métricas de los clientes.

## <a name="data-sources-and-destinations"></a>Orígenes y destinos de los datos
En la tabla siguiente se enumeran los tipos de datos que se pueden recopilar actualmente con el agente de Azure Monitor mediante reglas de recopilación de datos y a dónde se pueden enviar los datos. Consulte [¿Qué supervisa Azure Monitor?](../monitor-reference.md) para ver una lista con información detallada, soluciones y otros servicios que usan el agente de Azure Monitor para recopilar otros tipos de datos.


El agente de Azure Monitor envía datos a métricas de Azure Monitor o a un área de trabajo de Log Analytics que admite registros de Azure Monitor.

| Origen de datos | Destinations | Descripción |
|:---|:---|:---|
| Rendimiento        | Métricas de Azure Monitor<sup>1</sup><br>Área de trabajo de Log Analytics | Valores numéricos que miden el rendimiento de diferentes aspectos del sistema operativo y las cargas de trabajo. |
| Registros de eventos de Windows | Área de trabajo de Log Analytics | Información enviada al sistema de registro de eventos de Windows. |
| syslog             | Área de trabajo de Log Analytics | Información enviada al sistema de registro de eventos de Windows. |

<sup>1</sup> Actualmente hay una limitación en el agente de Azure Monitor para Linux por la que no se admite el uso de métricas de Azure Monitor como *único* destino. Su uso junto con los registros de Azure Monitor funciona. Esta limitación se solucionará en la siguiente actualización de la extensión.

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos
Consulte [Sistemas operativos admitidos](agents-overview.md#supported-operating-systems) para obtener una lista de las versiones de sistemas operativos Windows y Linux compatibles actualmente con el agente de Azure Monitor.



## <a name="security"></a>Seguridad
El agente de Azure Monitor no requiere ninguna clave, pero en su lugar requiere una [identidad administrada asignada por el sistema](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#system-assigned-managed-identity). Debe tener una identidad administrada asignada por el sistema habilitada en cada máquina virtual antes de implementar el agente.

## <a name="networking"></a>Redes
El agente de Azure Monitor admite etiquetas de servicio de Azure (se requieren tanto etiquetas AzureMonitor como AzureResourceManager), pero todavía no funciona en ámbitos de Private Link de Azure Monitor. Si la máquina se conecta mediante un servidor proxy para comunicarse a través de Internet, consulte los requisitos siguientes para comprender qué configuración de red es necesaria.

### <a name="proxy-configuration"></a>Configuración de proxy

Las extensiones del agente de Azure Monitor para Windows y Linux pueden comunicarse a través de un servidor proxy o una puerta de enlace de Log Analytics con Azure Monitor mediante el protocolo HTTPS (para máquinas virtuales de Azure, conjuntos de escalado de máquinas virtuales de Azure y Azure Arc para servidores). Esto se configura mediante las opciones de extensiones, como se indica a continuación, y admite la autenticación anónima y básica (nombre de usuario y contraseña).  

1. Use este diagrama de flujo simple para determinar primero los valores de los parámetros *setting* y *protectedSetting*:

   ![Diagrama de flujo para determinar los valores de los parámetros setting y protectedSetting al habilitar la extensión](media/azure-monitor-agent-overview/proxy-flowchart.png)


2. Una vez determinados los valores de los parámetros *setting* y *protectedSetting*, proporcione estos parámetros adicionales al implementar el agente de Azure Monitor mediante comandos de PowerShell (los siguientes ejemplos son para máquinas virtuales de Azure):

    | Parámetro | Value |
    |:---|:---|
    | SettingString | Objeto JSON del diagrama de flujo anterior, convertido en cadena; omita si no es aplicable. Ejemplo: {"proxy":{"mode":"application","address":"http://[address]:[port]","auth": false}} |
    | ProtectedSettingString | Objeto JSON del diagrama de flujo anterior, convertido en cadena; omita si no es aplicable. Ejemplo: {"proxy":{"username": "[username]","password": "[password]"}} |


# <a name="windows"></a>[Windows](#tab/PowerShellWindows)
```powershell
Set-AzVMExtension -ExtensionName AzureMonitorWindowsAgent -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -VMName <virtual-machine-name> -Location <location> -TypeHandlerVersion 1.0 -SettingString <settingString> -ProtectedSettingString <protectedSettingString>
```

# <a name="linux"></a>[Linux](#tab/PowerShellLinux)
```powershell
Set-AzVMExtension -ExtensionName AzureMonitorLinuxAgent -ExtensionType AzureMonitorLinuxAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -VMName <virtual-machine-name> -Location <location> -TypeHandlerVersion 1.5 -SettingString <settingString> -ProtectedSettingString <protectedSettingString>
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Instale el agente de Azure Monitor](azure-monitor-agent-install.md) en máquinas virtuales Windows y Linux.
- [Cree una regla de recopilación de datos](data-collection-rule-azure-monitor-agent.md) para recopilar datos del agente y enviarlos a Azure Monitor.
