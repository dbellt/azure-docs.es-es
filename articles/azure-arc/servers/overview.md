---
title: Información general de los servidores habilitados para Azure Arc
description: Aprenda a usar los servidores habilitados para Azure Arc para administrar las máquinas hospedadas fuera de Azure como un recurso de Azure.
keywords: azure automation, DSC, powershell, desired state configuration, update management, change tracking, inventory, runbooks, python, graphical, hybrid
ms.date: 05/26/2021
ms.topic: overview
ms.openlocfilehash: 2cf70cbf20d024d92a3a2025ca6b659ffdd8bffa
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112294686"
---
# <a name="what-is-azure-arc-enabled-servers"></a>¿Qué son los servidores habilitados para Azure Arc?

Los servidores habilitados para Azure Arc permiten administrar servidores físicos y máquinas virtuales Windows y Linux hospedados *fuera* de Azure, en la red corporativa o en otro proveedor de nube. Esta experiencia de administración está diseñada para ser coherente con el modo de administración de las máquinas virtuales nativas de Azure. Cuando una máquina híbrida se conecta a Azure, se convierte en una máquina conectada y se trata como un recurso de Azure. Cada máquina conectada tiene un identificador de recurso, se incluye en un grupo de recursos y se beneficia de las construcciones estándar de Azure como Azure Policy y la aplicación de etiquetas. Los proveedores de servicios que administran la infraestructura local de un cliente pueden administrar sus máquinas híbridas, tal como lo hacen actualmente con los recursos nativos de Azure en varios entornos de cliente, mediante [Azure Lighthouse](../../lighthouse/how-to/manage-hybrid-infrastructure-arc.md) con Azure Arc.

Para ofrecer esta experiencia con las máquinas híbridas hospedadas fuera de Azure, el agente de Azure Connected Machine debe estar instalado en cada una de las máquinas que planea conectar con Azure. Este agente no proporciona ninguna otra funcionalidad y no reemplaza al [agente de Log Analytics](../../azure-monitor/agents/log-analytics-agent.md). El agente de Log Analytics para Windows y Linux es necesario si desea supervisar de forma proactiva el sistema operativo y las cargas de trabajo que se ejecutan en la máquina, administrarlos mediante runbooks de Automation o soluciones como Update Management, o usar otros servicios de Azure como [Azure Security Center](../../security-center/security-center-introduction.md).

>[!NOTE]
> El [agente de Azure Monitor](../../azure-monitor/agents/azure-monitor-agent-overview.md) (AMA), que se encuentra actualmente en versión preliminar, no reemplaza al agente de Connected Machine. El agente de Azure Monitor reemplazará al agente de Log Analytics, a la extensión Diagnostics y al agente de Telegraf para máquinas Windows y Linux. Revise la documentación de Azure Monitor sobre el nuevo agente para obtener más detalles.

## <a name="supported-scenarios"></a>Escenarios admitidos

Al conectar una máquina a los servidores habilitados para Azure Arc, se habilita la capacidad de realizar las siguientes tareas de administración y supervisión de la configuración:
- Asigne [configuraciones de invitado de Azure Policy](../../governance/policy/concepts/guest-configuration.md) de la misma manera que en la asignación de directivas para máquinas virtuales de Azure. En la actualidad, la mayoría de las directivas de configuración de invitado no aplican configuraciones, solo auditan la configuración dentro de la máquina. Para comprender el costo de usar las directivas de configuración de Azure Policy con servidores habilitados para Arc, consulte la [guía de precios](https://azure.microsoft.com/pricing/details/azure-policy/) de Azure Policy.

- Informe de los cambios de configuración del software instalado, los servicios de Microsoft, el Registro y los archivos de Windows y los demonios de Linux en los servidores supervisados mediante [Seguimiento de cambios e inventario](../../automation/change-tracking/overview.md) de Azure Automation y [Supervisión de la integridad de los archivos de Azure Security Center](../../security-center/security-center-file-integrity-monitoring.md), en servidores habilitados con [Azure Defender para servidores](../../security-center/defender-for-servers-introduction.md).

- Supervise el rendimiento del sistema operativo invitado de la máquina conectada y detecte los componentes de la aplicación para supervisar sus procesos y dependencias con otros recursos con los que la aplicación se comunica mediante [Insights para VM](../../azure-monitor/vm/vminsights-overview.md).

- Simplifique la implementación con otros servicios de Azure, como el área de trabajo de Log Analytics de Azure Monitor y [State Configuration](../../automation/automation-dsc-overview.md) de Azure Automation, con las [extensiones de máquina virtual de Azure](manage-vm-extensions.md) admitidas para la máquina Windows o Linux que no sea de Azure. Esto incluye realizar la instalación de software o la configuración posterior a la implementación mediante la extensión de script personalizado.

- Use [Update Management](../../automation/update-management/overview.md) en Azure Automation para administrar las actualizaciones del sistema operativo de los servidores Windows y Linux.

    > [!NOTE]
    > En este momento, no se admite la habilitación de Update Management directamente desde un servidor habilitado para Arc. Consulte [Habilitación de Update Management desde una cuenta de Automation](../../automation/update-management/enable-from-automation-account.md) para saber más sobre los requisitos y cómo habilitarlo para el servidor.

- Incluya los servidores que no son de Azure para la detección de amenazas avanzado y supervisar proactivamente las posibles amenazas de seguridad mediante [Azure Security Center](../../security-center/security-center-introduction.md) o [Azure Defender](../../security-center/azure-defender.md).

- Proteja servidores que no son de Azure con [Microsoft Defender para punto de conexión](/microsoft-365/security/defender-endpoint/microsoft-defender-endpoint), incluido mediante [Azure Defender](../../security-center/azure-defender.md), para la detección de amenazas, para la administración de vulnerabilidades y para supervisar proactivamente las posibles amenazas de seguridad.

Los datos de registro recopilados y almacenados en un área de trabajo de Log Analytics desde la máquina híbrida ahora contienen propiedades específicas de la máquina, como un identificador de recurso. Se puede usar para admitir el acceso al registro [resource-context](../../azure-monitor/logs/design-logs-deployment.md#access-mode).

[!INCLUDE [azure-lighthouse-supported-service](../../../includes/azure-lighthouse-supported-service.md)]

## <a name="supported-regions"></a>Regiones admitidas

Para obtener una lista definitiva de las regiones admitidas con servidores habilitados para Azure Arc, consulte la página [Productos de Azure por región](https://azure.microsoft.com/global-infrastructure/services/?products=azure-arc).

En la mayoría de los casos, la ubicación que seleccione al crear el script de instalación debe ser la región de Azure más cercana geográficamente a la ubicación de la máquina. Los datos en reposo se almacenan en la ubicación geográfica de Azure que contiene la región que especifique, lo que también puede afectar a su elección de región si tiene requisitos de residencia de datos. Si la región de Azure a la que está conectada la máquina experimente una interrupción, la máquina conectada no se verá afectada, aunque es posible que las operaciones de administración que usan Azure no se puedan completar. Si se produce una interrupción regional y se dispone de varias ubicaciones que admiten un servicio con redundancia geográfica, es mejor conectar las máquinas de cada ubicación a otra región de Azure.

La siguiente información de metadatos sobre la máquina conectada se recopila y almacena en la región donde se configura el recurso de la máquina de Azure Arc:

- Nombre y versión del sistema operativo
- Nombre del equipo
- Nombre de dominio completo (FQDN) del equipo
- Versión del agente Connected Machine

Por ejemplo, si la máquina se registra en Azure Arc en la región Este de EE. UU., estos datos se almacenan en la región de EE. UU.

### <a name="supported-environments"></a>Entornos admitidos

Los servidores habilitados para Arc permiten la administración de servidores físicos y máquinas virtuales hospedados *fuera* de Azure. Para obtener detalles concretos sobre qué entornos de nube híbrida que hospedan máquinas virtuales se admiten, vea [Requisitos previos del agente de Azure Connected Machine](agent-overview.md#supported-environments).

> [!NOTE]
> Los servidores habilitados para Arc no están diseñados para la administración de máquinas virtuales que se ejecutan en Azure, ni la permiten.

### <a name="agent-status"></a>Estado del agente

El agente Connected Machine envía un mensaje de latido al servicio cada 5 minutos. Si el servicio deja de recibir estos mensajes de latido de una máquina, esa máquina se considera sin conexión y el estado cambiará automáticamente a **Desconectado** en el portal en un plazo de 15 a 30 minutos. Al recibir un mensaje de latido subsiguiente del agente de Connected Machine, su estado se cambiará automáticamente a **Conectado**.

## <a name="next-steps"></a>Pasos siguientes

Antes de evaluar o habilitar los servidores habilitados para Arc en varias máquinas híbridas, consulte el artículo [Información general del agente de Connected Machine](agent-overview.md) a fin de conocer los requisitos, los detalles técnicos del agente y los métodos de implementación.
