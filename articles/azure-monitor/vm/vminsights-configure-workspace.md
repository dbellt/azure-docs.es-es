---
title: Configuración del área de trabajo de Log Analytics para VM Insights
description: Describe cómo crear y configurar el área de trabajo Log Analytics que usa VM Insights.
ms.topic: conceptual
ms.custom: references_regions, devx-track-azurepowershell
author: bwren
ms.author: bwren
ms.date: 12/22/2020
ms.openlocfilehash: 0a4b6e1f90e24b3289b453e849b4ad979b10e202
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108319924"
---
# <a name="configure-log-analytics-workspace-for-vm-insights"></a>Configuración del área de trabajo de Log Analytics para VM Insights
VM Insights recopila sus datos de una o varias áreas de trabajo de Log Analytics en Azure Monitor. Antes de incorporar agentes, debe crear y configurar un área de trabajo. En este artículo se describen los requisitos del área de trabajo y cómo configurarla para VM Insights.

## <a name="overview"></a>Información general
Una sola suscripción puede usar cualquier número de áreas de trabajo en función de sus requisitos. El único requisito del área de trabajo es que se encuentre en una ubicación admitida y esté configurada con la solución *VMInsights*.

Una vez configurada el área de trabajo, puede usar cualquiera de las opciones disponibles para instalar los agentes necesarios en la máquina virtual y el conjunto de escalado de máquinas virtuales, y especificar un área de trabajo para que envíen sus datos. VM Insights recopilará datos de cualquier área de trabajo configurada en su suscripción.

> [!NOTE]
> Al habilitar VM Insights en una única máquina virtual o en una instancia de Virtual Machine Scale Sets con Azure Portal, se le ofrece la opción de seleccionar un área de trabajo existente o crear una nueva. La solución *VMInsights* se instalará en este área de trabajo si aún no lo está. Después, puede usar este área de trabajo para otros agentes.


## <a name="create-log-analytics-workspace"></a>Creación de un área de trabajo de Log Analytics

>[!NOTE]
>La información que se describe en esta sección también se aplica a la [solución Service Map](service-map.md). 

Acceda a las áreas de trabajo de Log Analytics en Azure Portal desde el menú **Áreas de trabajo de Log Analytics**.

[![Áreas de trabajo de Log Analytics](media/vminsights-configure-workspace/log-analytics-workspaces.png)](media/vminsights-configure-workspace/log-analytics-workspaces.png#lightbox)

Puede crear una nueva área de trabajo de Log Analytics mediante cualquiera de los métodos siguientes. Consulte [Diseño de la implementación de registros de Azure Monitor](../logs/design-logs-deployment.md) para obtener instrucciones sobre cómo determinar el número de áreas de trabajo que debe usar en su entorno y cómo diseñar su estrategia de acceso.


* [Azure Portal](../logs/quick-create-workspace.md)
* [CLI de Azure](../logs/quick-create-workspace-cli.md)
* [PowerShell](../logs/powershell-workspace-configuration.md)
* [Azure Resource Manager](../logs/resource-manager-workspace.md)

## <a name="supported-regions"></a>Regiones admitidas
VM Insights admite un área de trabajo de Log Analytics en cualquiera de las [regiones compatibles con Log Analytics](https://azure.microsoft.com/global-infrastructure/services/?products=monitor&regions=all) excepto las siguientes:

- Centro-oeste de Alemania
- Centro de Corea del Sur

>[!NOTE]
>Puede supervisar máquinas virtuales de Azure en cualquier región. Las máquinas virtuales no se limitan a las regiones admitidas por el área de trabajo de Log Analytics.

## <a name="azure-role-based-access-control"></a>Control de acceso basado en roles de Azure
Para habilitar las características de VM Insights y acceder a estas, debe tener el rol de [Colaborador de Log Analytics](../logs/manage-access.md#manage-access-using-azure-permissions) en el área de trabajo. Para ver los datos de rendimiento, mantenimiento y el mapa, debe tener el [rol de lector de supervisión](../roles-permissions-security.md#built-in-monitoring-roles) en la máquina virtual de Azure. Para más información acerca de cómo controlar el acceso a un área de trabajo de Log Analytics, consulte [Administración de áreas de trabajo](../logs/manage-access.md).

## <a name="add-vminsights-solution-to-workspace"></a>Incorporación de la solución VMInsights al área de trabajo
Para poder utilizar un área de trabajo de Log Analytics con VM Insights, debe tener instalada la solución *VMInsights*. Los métodos para configurar el área de trabajo se describen en las secciones siguientes.

> [!NOTE]
> Al agregar la solución *VMInsights* al área de trabajo, todas las máquinas virtuales existentes conectadas al área de trabajo comenzarán a enviar datos a InsightsMetrics. Los datos del resto de tipos de datos no se recopilarán hasta que agregue Dependency Agent a las máquinas virtuales existentes conectadas al área de trabajo.

### <a name="azure-portal"></a>Azure portal
Existen tres opciones para configurar un área de trabajo existente mediante Azure Portal. A continuación se describe cada uno.

Para configurar una sola área de trabajo, vaya a la opción **Máquinas virtuales** en el menú **Azure Monitor**, seleccione **Other onboarding options** (Otras opciones de incorporación) y, a continuación, **Configurar un área de trabajo**. Seleccione una suscripción y un área de trabajo y, a continuación, haga clic en **Configurar**.

[![Configuración del área de trabajo](../vm/media/vminsights-enable-policy/configure-workspace.png)](../vm/media/vminsights-enable-policy/configure-workspace.png#lightbox)

Para configurar varias áreas de trabajo, seleccione la pestaña **Configuración del área de trabajo** en el menú **Máquinas virtuales** del menú **Supervisión** de Azure Portal. Establezca los valores de filtro para mostrar una lista de áreas de trabajo existentes. Marque la casilla situada junto a cada área de trabajo que quiera habilitar y, a continuación, haga clic en **Configurar selección**.

[![Configuración del área de trabajo](../vm/media/vminsights-enable-policy/workspace-configuration.png)](../vm/media/vminsights-enable-policy/workspace-configuration.png#lightbox)


Al habilitar VM Insights en una única máquina virtual o en una instancia de Virtual Machine Scale Sets con Azure Portal, se le ofrece la opción de seleccionar un área de trabajo existente o crear una nueva. La solución *VMInsights* se instalará en este área de trabajo si aún no lo está. Después, puede usar este área de trabajo para otros agentes.

[![Habilitación de una única VM en el portal](../vm/media/vminsights-enable-portal/enable-vminsights-vm-portal.png)](../vm/media/vminsights-enable-portal/enable-vminsights-vm-portal.png#lightbox)


### <a name="resource-manager-template"></a>Plantilla de Resource Manager
Las plantillas de Azure Resource Manager para VM Insights se suministran en un archivo (.zip) que puede [descargar desde nuestro repositorio de GitHub](https://aka.ms/VmInsightsARMTemplates). Incluye una plantilla llamada **ConfigureWorkspace** que configura un área de trabajo de Log Analytics para VM Insights. Esta plantilla se implementa mediante cualquiera de los métodos estándar, incluidos los comandos de PowerShell y de la CLI de ejemplo siguientes: 

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az deployment group create --name ConfigureWorkspace --resource-group my-resource-group --template-file CreateWorkspace.json  --parameters workspaceResourceId='/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace' workspaceLocation='eastus'

```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
New-AzResourceGroupDeployment -Name ConfigureWorkspace -ResourceGroupName my-resource-group -TemplateFile ConfigureWorkspace.json -workspaceResourceId /subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace -location eastus
```

---



## <a name="next-steps"></a>Pasos siguientes
- Consulte [Incorporación de agentes a VM Insights](vminsights-enable-overview.md) para conectar agentes a VM Insights.
- Consulte [Soluciones de supervisión como destino en Azure Monitor (versión preliminar)](../insights/solution-targeting.md) para limitar la cantidad de datos que se envían desde una solución al área de trabajo.
