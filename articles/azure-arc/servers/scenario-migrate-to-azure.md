---
title: Migración de servidores habilitados para Azure Arc a Azure
description: Obtenga información sobre cómo migrar a Azure servidores habilitados para Azure Arc que se ejecutan en el entorno local u otro entorno en la nube.
ms.date: 05/06/2021
ms.topic: conceptual
ms.openlocfilehash: 4c8a5ea1974e251ba147fb7ee09b9ac49c364876
ms.sourcegitcommit: c1b0d0b61ef7635d008954a0d247a2c94c1a876f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109629499"
---
# <a name="migrate-your-on-premises-or-other-cloud-arc-enabled-server-to-azure"></a>Migración de un servidor local o en la nube habilitado para Arc a Azure

Este artículo está pensado para ayudarle a planificar y migrar correctamente a Azure un servidor local o una máquina virtual administrados por servidores habilitados para Azure Arc. Al seguir estos pasos, puede realizar la transición de la administración desde servidores habilitados para Arc en función de las extensiones de VM admitidas instaladas y los servicios de Azure en función de su identidad de recursos de servidor de Arc.

Antes de seguir estos pasos, revise el artículo [Preparación de las máquinas locales para la migración a Azure](../../migrate/prepare-for-migration.md) de Azure Migrate para comprender los requisitos de preparación para el uso de Azure Migrate.

En este artículo:

* Hará un inventario de las extensiones instaladas de VM compatibles con servidores habilitados para Azure Arc.
* Desinstalará todas las extensiones de VM del servidor habilitado para Arc.
* Identificará los servicios de Azure configurados para autenticarse con la identidad administrada por el servidor habilitado para Arc, y se preparará para actualizar esos servicios para usar la identidad de VM de Azure después de la migración.
* Revisará los derechos de acceso del control de acceso basado en roles de Azure (Azure RBAC) concedidos al recurso de servidor habilitado para Arc para mantener quién tiene acceso al recurso después de haber migrado a una VM de Azure. 
* Eliminará de Azure la identidad de recursos del servidor habilitado para Arc y quitará el agente del servidor habilitado para Arc.
* Instalará el agente invitado de Azure.
* Migrará el servidor o la VM a Azure.

## <a name="step-1-inventory-and-remove-vm-extensions"></a>Paso 1: Hacer un inventario de las extensiones de VM y eliminarlas

Para hacer un inventario de las extensiones de VM instaladas en el servidor habilitado para Arc, puede enumerarlas mediante la CLI de Azure o con Azure PowerShell.

Con Azure PowerShell, use el comando [Get-AzConnectedMachineExtension](/powershell/module/az.connectedmachine/get-azconnectedmachineextension) con los parámetros `-MachineName` y `-ResourceGroupName`.

Con la CLI de Azure, use el comando [az connectedmachine extension list](/cli/azure/ext/connectedmachine/connectedmachine/extension#ext_connectedmachine_az_connectedmachine_extension_list) con los parámetros `--machine-name` y `--resource-group`. De forma predeterminada, la salida de los comandos de la CLI de Azure está en JSON (notación de objetos JavaScript). Para cambiar la salida predeterminada a una lista o tabla, por ejemplo, use [az configure --output](/cli/azure/reference-index). También puede agregar `--output` a cualquier comando para un cambio específico del formato de salida.

Después de detectar qué extensiones de VM hay implementadas, puede quitarlas mediante [Azure Portal](manage-vm-extensions-portal.md), [Azure PowerShell](manage-vm-extensions-powershell.md) o la [CLI de Azure](manage-vm-extensions-cli.md). Si la extensión de VM de Log Analytics o la extensión de VM de Dependency Agent se implementaron mediante Azure Policy y la [iniciativa de VM Insights](../../azure-monitor/vm/vminsights-enable-policy.md), es necesario [crear una exclusión](../../governance/policy/tutorials/create-and-manage.md#remove-a-non-compliant-or-denied-resource-from-the-scope-with-an-exclusion) para evitar la reevaluación y la implementación de las extensiones en el servidor habilitado para Arc antes de que se complete la migración.

## <a name="step-2-review-access-rights"></a>Paso 2: Revisar los derechos de acceso 

Enumere las asignaciones de roles para el recurso de servidores habilitados para Arc, mediante [Azure PowerShell](../../role-based-access-control/role-assignments-list-powershell.md#list-role-assignments-for-a-resource), y con otro código de PowerShell puede exportar los resultados a CSV u otro formato. 

Si usa una identidad administrada para una aplicación o un proceso que se ejecutan en un servidor habilitado para Arc, tiene que asegurarse de que la VM de Azure tenga asignada una identidad administrada. Para ver la asignación de roles para una identidad administrada, puede usar el cmdlet `Get-AzADServicePrincipal` de Azure PowerShell. Para más información, consulte [Lista de asignaciones de roles para una identidad administrada](../../role-based-access-control/role-assignments-list-powershell.md#list-role-assignments-for-a-managed-identity). 

También se usa una identidad administrada por el sistema cuando se usa Azure Policy para auditar la configuración dentro de una máquina o servidor. Con los servidores habilitados para Arc, se incluye el agente de Guest Configuration y se realiza la validación de la configuración de auditoría. Después de la migración, consulte [Requisitos de implementación de Azure Virtual Machines](../../governance/policy/concepts/guest-configuration.md#deploy-requirements-for-azure-virtual-machines) para obtener información sobre cómo configurar la VM de Azure manualmente o con la directiva con la extensión de Guest Configuration.

Actualice la asignación de roles con los recursos a los que tiene acceso la identidad administrada para permitir que la nueva identidad de VM de Azure se autentique en esos servicios. Consulte a continuación para obtener información sobre [cómo funcionan las identidades administradas en los recursos de Azure para una máquina virtual (VM) de Azure](../../active-directory/managed-identities-azure-resources/how-managed-identities-work-vm.md).

## <a name="step-3-disconnect-from-azure-arc-and-uninstall-agent"></a>Paso 3: Desconectarse de Azure Arc y desinstalar el agente

Elimine el identificador de recurso del servidor habilitado para Arc en Azure siguiendo uno de los métodos a continuación:

   * Ejecutar el comando `azcmagent disconnect` en la máquina o el servidor.

   * Del servidor habilitado para Arc registrado y seleccionado en Azure Portal, al seleccionar **Eliminar** en la barra superior.

   * Mediante la [CLI de Azure](../../azure-resource-manager/management/delete-resource-group.md?tabs=azure-cli#delete-resource) o [Azure PowerShell](../../azure-resource-manager/management/delete-resource-group.md?tabs=azure-powershell#delete-resource). En el caso del parámetro `ResourceType`, use `Microsoft.HybridCompute/machines`.

A continuación, quite el agente de Windows o Linux de los servidores habilitados para Azure Arc según los pasos de [Eliminación del agente](manage-agent.md#remove-the-agent).

## <a name="step-4-install-the-azure-guest-agent"></a>Paso 4: Instalar el agente invitado de Azure

La VM que se migra a Azure desde el entorno local no tiene instalado el agente invitado Linux ni Windows de Azure. En estos casos, tendrá que instalar manualmente el agente de máquina virtual. Para más información sobre cómo instalar el agente de VM, consulte [Información general del agente Windows de máquina virtual de Azure](../../virtual-machines/extensions/agent-windows.md) o [Información general del agente Linux de máquina virtual de Azure](../../virtual-machines/extensions/agent-linux.md).

## <a name="step-5-migrate-server-or-machine-to-azure"></a>Paso 5: Migrar un servidor o una máquina a Azure

Antes de continuar con la migración con Azure Migration, revise el artículo [Preparación de las máquinas locales para la migración a Azure](../../migrate/prepare-for-migration.md) para obtener información sobre los requisitos necesarios para usar Azure Migrate. Para completar la migración a Azure, revise las [opciones de migración](../../migrate/prepare-for-migration.md#next-steps) de Azure Migrate en función de su entorno.

## <a name="step-6-deploy-azure-vm-extensions"></a>Paso 6: Implementar las extensiones de VM de Azure

Después de la migración y de completar todos los pasos de configuración posteriores a la migración, puede implementar las extensiones de VM de Azure en función de las extensiones de VM instaladas originalmente en el servidor habilitado para Arc. Revise [Características y extensiones de las máquinas virtuales de Azure](../../virtual-machines/extensions/overview.md) para ayudar a planificar la implementación de las extensiones. 

Para reanudar el uso de la configuración de auditoría dentro de una máquina definiciones de la directiva de Guest Configuration de Azure Policy, consulte [Habilitar configuración de invitado](../../governance/policy/concepts/guest-configuration.md#enable-guest-configuration).

Si la extensión de VM de Log Analytics o la extensión de VM de Dependency Agent se implementaron mediante Azure Policy y la [iniciativa VM Insights](../../azure-monitor/vm/vminsights-enable-policy.md), quite la [exclusión](../../governance/policy/tutorials/create-and-manage.md#remove-a-non-compliant-or-denied-resource-from-the-scope-with-an-exclusion) que creó anteriormente. Para usar Azure Policy para habilitar las máquinas virtuales de Azure, consulte [Implementación de Azure Monitor a escala mediante Azure Policy](../../azure-monitor/deploy-scale.md#vm-insights). 

## <a name="next-steps"></a>Pasos siguientes

Puede encontrar información sobre la solución de problemas en la guía [Solución de problemas de conexión del agente de Connected Machine](troubleshoot-agent-onboard.md).