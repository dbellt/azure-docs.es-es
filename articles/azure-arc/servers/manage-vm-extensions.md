---
title: Administración de extensiones de máquina virtual con servidores habilitados para Azure Arc
description: Los servidores habilitados para Azure Arc pueden administrar la implementación de extensiones de máquina virtual que proporcionan tareas de automatización y configuración posteriores a la implementación con máquinas virtuales que no son de Azure.
ms.date: 04/13/2021
ms.topic: conceptual
ms.openlocfilehash: e28cd7753fc85f2e40385c65392fea73502aa05b
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107832849"
---
# <a name="virtual-machine-extension-management-with-azure-arc-enabled-servers"></a>Administración de extensiones de máquina virtual con servidores habilitados para Azure Arc

Las extensiones de máquina virtual (VM) son pequeñas aplicaciones que realizan tareas de automatización y configuración posteriores a la implementación en máquinas virtuales de Azure. Por ejemplo, si una máquina virtual necesita que se instale software, protección antivirus o ejecutar un script en ella, se puede usar una extensión de máquina virtual.

Los servidores habilitados para Azure Arc permiten implementar extensiones de máquina virtual de Azure en máquinas virtuales Windows y Linux que no son de Azure, lo que simplifica la administración de la máquina híbrida local durante su ciclo de vida. Las extensiones de VM se pueden administrar con los siguientes métodos en las máquinas híbridas o servidores administrados por servidores habilitados para Arc:

- [Azure Portal](manage-vm-extensions-portal.md)
- La [CLI de Azure](manage-vm-extensions-cli.md)
- [Azure PowerShell](manage-vm-extensions-powershell.md)
- [Plantillas de Azure Resource Manager](manage-vm-extensions-template.md)

> [!NOTE]
> Los servidores habilitados de Azure Arc no admiten la implementación y administración de extensiones de máquina virtual en máquinas virtuales de Azure. Para las máquinas virtuales de Azure, consulte el siguiente artículo de [información general sobre la extensión de máquina virtual](../../virtual-machines/extensions/overview.md).

## <a name="key-benefits"></a>Ventajas principales

La compatibilidad con la extensión de máquina virtual de servidores habilitados para Azure Arc ofrece las ventajas principales siguientes:

- Recopile datos de registro para analizarlos con los [registros de Azure Monitor](../../azure-monitor/logs/data-platform-logs.md) mediante la habilitación de la extensión de máquina virtual del agente de Log Analytics. Esto es especialmente útil para llevar a cabo un análisis complejo de datos procedentes de diferentes tipos de orígenes.

- Con [Información de máquinas virtuales](../../azure-monitor/vm/vminsights-overview.md), analiza el rendimiento de las VM Windows y Linux, y supervisa sus procesos y dependencias en otros recursos y procesos externos. Esto se logra al habilitar las extensiones de máquina virtual del agente de Log Analytics y Dependency Agent.

- Descargue y ejecute scripts en máquinas conectadas híbridas mediante la extensión de script personalizado. Esta extensión es útil para la configuración posterior a la implementación, la instalación de software o cualquier otra tarea de configuración o administración.

- Actualización automática de los certificados almacenados en almacén de [Azure Key Vault](../../key-vault/general/overview.md).

## <a name="availability"></a>Disponibilidad

La funcionalidad de la extensión de máquina virtual solo está disponible en la lista de [regiones admitidas](overview.md#supported-regions). Asegúrese de incorporar el equipo en una de estas regiones.

## <a name="extensions"></a>Extensiones

En esta versión se admiten las siguientes extensiones de máquina virtual en máquinas Windows y Linux.

Para obtener información sobre el paquete del agente de Azure Connected Machine y los detalles del componente del agente de extensión, consulte [Información general del agente](agent-overview.md#agent-component-details).

> [!NOTE]
> Recientemente se quitó la compatibilidad con la extensión de máquina virtual de DSC de los servidores habilitados para ARC. Como alternativa, se recomienda usar la extensión de script personalizado para administrar la configuración posterior a la implementación del servidor o la máquina.

### <a name="windows-extensions"></a>Extensiones de Windows

|Extensión |Publicador |Tipo |Información adicional |
|----------|----------|-----|-----------------------|
|Detector de vulnerabilidades integrado de Azure Defender |Qualys |WindowsAgent.AzureSecurityCenter |[Solución de evaluación de vulnerabilidades integrada en Azure Defender para Azure y máquinas híbridas](../../security-center/deploy-vulnerability-assessment-vm.md)|
|Extensión Custom Script |Microsoft.Compute | CustomScriptExtension |[Extensión de script personalizado de Windows](../../virtual-machines/extensions/custom-script-windows.md)|
|Agente de Log Analytics |Microsoft.EnterpriseCloud.Monitoring |MicrosoftMonitoringAgent |[Extensión de máquina virtual de Log Analytics para Windows](../../virtual-machines/extensions/oms-windows.md)|
|Azure Monitor para VM (información detallada) |Microsoft.Azure.Monitoring.DependencyAgent |DependencyAgentWindows | [Extensión de máquina virtual de Dependency Agent para Windows](../../virtual-machines/extensions/agent-dependency-windows.md)|
|Sincronización del certificado de Azure Key Vault | Microsoft.Azure.Key.Vault |KeyVaultForWindows | [Extensión de máquina virtual de Key Vault para Windows](../../virtual-machines/extensions/key-vault-windows.md) |
|Agente de Azure Monitor |Microsoft.Azure.Monitor |AzureMonitorWindowsAgent |[Instalación del agente de Azure Monitor (versión preliminar)](../../azure-monitor/agents/azure-monitor-agent-install.md) |

### <a name="linux-extensions"></a>Extensiones de Linux

|Extensión |Publicador |Tipo |Información adicional |
|----------|----------|-----|-----------------------|
|Detector de vulnerabilidades integrado de Azure Defender |Qualys |LinuxAgent.AzureSecurityCenter |[Solución de evaluación de vulnerabilidades integrada en Azure Defender para Azure y máquinas híbridas](../../security-center/deploy-vulnerability-assessment-vm.md)|
|Extensión Custom Script |Microsoft.Azure.Extensions |CustomScript |[Extensión de script personalizado de Linux versión 2](../../virtual-machines/extensions/custom-script-linux.md) |
|Agente de Log Analytics |Microsoft.EnterpriseCloud.Monitoring |OmsAgentForLinux |[Extensión de máquina virtual de Log Analytics para Linux](../../virtual-machines/extensions/oms-linux.md) |
|Azure Monitor para VM (información detallada) |Microsoft.Azure.Monitoring.DependencyAgent |DependencyAgentLinux |[Extensión de máquina virtual de Dependency Agent para Linux](../../virtual-machines/extensions/agent-dependency-linux.md) |
|Sincronización del certificado de Azure Key Vault | Microsoft.Azure.Key.Vault |KeyVaultForLinux | [Extensión de máquina virtual de Key Vault para Linux](../../virtual-machines/extensions/key-vault-linux.md) |
|Agente de Azure Monitor |Microsoft.Azure.Monitor |AzureMonitorLinuxAgent |[Instalación del agente de Azure Monitor (versión preliminar)](../../azure-monitor/agents/azure-monitor-agent-install.md) |

## <a name="prerequisites"></a>Requisitos previos

Esta característica depende de los siguientes proveedores de recursos de Azure en su suscripción:

- **Microsoft.HybridCompute**
- **Microsoft.GuestConfiguration**

Si aún no están registrados, siga los pasos descritos en [Registro de proveedores de recursos de Azure](agent-overview.md#register-azure-resource-providers).

No olvide revisar la documentación de cada extensión de máquina virtual a la que se hace referencia en la tabla anterior para saber si tiene algún requisito de red o del sistema. Esto le ayudará a evitar que se produzcan problemas de conectividad con un servicio o característica de Azure que se base en esa extensión de máquina virtual.

### <a name="log-analytics-vm-extension"></a>Extensión de VM de Log Analytics

La extensión de máquina virtual del agente de Log Analytics para Linux requiere que Python 2.x esté instalado en el equipo de destino.

### <a name="azure-key-vault-vm-extension-preview"></a>Extensión de máquina virtual de Azure Key Vault (versión preliminar)

La extensión de máquina virtual de Key Vault (versión preliminar) no es compatible con los siguientes sistemas operativos Linux:

- CentOS Linux 7 (x64)
- Red Hat Enterprise Linux (RHEL) 7 (x64)
- Amazon Linux 2 (x64)

La implementación de la extensión de máquina virtual de Key Vault (versión preliminar) solo se admite mediante:

- La CLI de Azure
- Azure PowerShell
- Plantilla del Administrador de recursos de Azure

Antes de implementar la extensión, debe completar lo siguiente:

1. [Cree un almacén y un certificado](../../key-vault/certificates/quick-create-portal.md) (autofirmado o importado).

2. Conceda al servidor habilitado para Azure Arc acceso al secreto del certificado. Si usa la [versión preliminar de RBAC](../../key-vault/general/rbac-guide.md), busque el nombre del recurso de Azure Arc y asígnele el rol **Usuario de secretos de almacén de claves (versión preliminar)** . Si está usando una [directiva de acceso de Key Vault](../../key-vault/general/assign-access-policy-portal.md), asigne los permisos para **obtener** secretos a la identidad asignada por el sistema del recurso de Azure Arc.

### <a name="connected-machine-agent"></a>Agente de Connected Machine

Compruebe que el equipo coincida con las [versiones compatibles](agent-overview.md#supported-operating-systems) de los sistemas operativos Windows y Linux para el agente de Azure Connected Machine.

La versión mínima del agente de Connected Machine que es compatible con esta característica en Windows y Linux es la 1.0.

Para actualizar la máquina a la versión requerida del agente, consulte [Actualización del agente](manage-agent.md#upgrading-agent).

## <a name="next-steps"></a>Pasos siguientes

Puede implementar, administrar y quitar extensiones de VM con la [CLI de Azure](manage-vm-extensions-cli.md), [Azure PowerShell](manage-vm-extensions-powershell.md), [Azure Portal](manage-vm-extensions-portal.md) o [plantillas de Azure Resource Manager](manage-vm-extensions-template.md).
