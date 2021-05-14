---
title: Soluciones de supervisión en Azure Monitor | Microsoft Docs
description: Las soluciones de supervisión de Azure Monitor son una colección de reglas de lógica, visualización y adquisición de datos que proporcionan métricas que giran en torno a una determinada área de problemas.  En este artículo se proporciona información sobre la instalación y el uso de soluciones de supervisión.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/16/2020
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 6a98571f513e25d801acd4f4a9c2901dcd56fabc
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107869228"
---
# <a name="monitoring-solutions-in-azure-monitor"></a>Soluciones de supervisión en Azure Monitor

Las soluciones de supervisión de Azure Monitor proporcionan análisis del funcionamiento de un servicio o una aplicación de Azure en particular. En este artículo se proporciona una breve descripción de las soluciones de supervisión en Azure y los detalles sobre su uso e instalación. Puede agregar soluciones de supervisión a su instancia de Azure Monitor para todas las aplicaciones y los servicios que use. Normalmente están disponibles sin costo, excepto la recopilación de datos que podría suponer cargos por uso.

## <a name="use-monitoring-solutions"></a>Uso de soluciones de supervisión

La página de **información general** de soluciones de Azure Monitor muestra un icono para cada solución instalada en un área de trabajo de Log Analytics. Para abrir esta página, vaya a **Azure Monitor** en [Azure Portal](https://ms.portal.azure.com). En el menú **Conclusiones**, seleccione **Más** para abrir el **Centro de conclusiones** y haga clic en **Áreas de trabajo de Log Analytics**.

[![Centro de conclusiones](media/solutions/insights-hub.png)](media/solutions/insights-hub.png#lightbox)


Use los cuadros de lista desplegable en la parte superior de la pantalla para cambiar el área de trabajo o el intervalo de tiempo usado para los iconos. Haga clic en el icono de una solución para abrir la vista que incluye un análisis más detallado de los datos recopilados.

[![Captura de pantalla que muestra el menú de Azure Portal con soluciones seleccionadas y soluciones mostradas en el panel Soluciones.](media/solutions/overview.png)](media/solutions/overview.png#lightbox)

Las soluciones de supervisión pueden contener varios tipos de recursos de Azure; puede ver todos los recursos que incluyen las soluciones igual que cualquier otro recurso. Por ejemplo, las consultas de registro incluidas en la solución se muestran en **Consultas de la solución** en el [Explorador de consultas](../logs/log-analytics-tutorial.md). Puede utilizar estas consultas al realizar un análisis ad hoc con [consultas de registros](../logs/log-query-overview.md).

## <a name="list-installed-monitoring-solutions"></a>Lista de soluciones de supervisión instaladas

### <a name="portal"></a>[Portal](#tab/portal)

Utilice el procedimiento siguiente para enumerar las soluciones de supervisión instaladas en su suscripción.

1. Vaya a [Azure Portal](https://ms.portal.azure.com). Busque y seleccione **Soluciones**.
1. Se mostrarán las soluciones instaladas en todas las áreas de trabajo. Al nombre de la solución le sigue el nombre del área de trabajo donde está instalada.
1. Use los cuadros de lista desplegable de la parte superior de la pantalla para filtrar por suscripción o grupo de recursos.

![Lista de todas las soluciones](media/solutions/list-solutions-all.png)

Haga clic en el nombre de una solución para abrir su página de resumen. Esta página muestra las vistas incluidas en la solución y proporciona diferentes opciones para la solución y su área de trabajo. Para ver la página de resumen de una solución, siga uno de los procedimientos descritos anteriormente para mostrar las soluciones y haga clic en el nombre concreto.

![Propiedades de la solución](media/solutions/solution-properties.png)

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el comando [az monitor log-analytics solution list](/cli/azure/monitor/log-analytics/solution#az_monitor_log_analytics_solution_list) para enumerar las soluciones de supervisión instaladas en la suscripción.   Antes de ejecutar el comando `list`, siga los requisitos previos que se indican en [Instalación de una solución de supervisión](#install-a-monitoring-solution).

```azurecli
# List all log-analytics solutions in the current subscription.
az monitor log-analytics solution list

# List all log-analytics solutions for a specific subscription
az monitor log-analytics solution list --subscription MySubscription

# List all log-analytics solutions in a resource group
az monitor log-analytics solution list --resource-group MyResourceGroup
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Use el cmdlet [Get-AzMonitorLogAnalyticsSolution](/powershell/module/az.monitoringsolutions/get-azmonitorloganalyticssolution) para mostrar las soluciones de supervisión instaladas en la suscripción. Antes de ejecutar estos comandos, siga los requisitos previos que se indican en [Instalación de una solución de supervisión](#install-a-monitoring-solution).

```azurepowershell-interactive
# List all log-analytics solutions in the current subscription.
Get-AzMonitorLogAnalyticsSolution

# List all log-analytics solutions for a specific subscription
Get-AzMonitorLogAnalyticsSolution -SubscriptionId 00000000-0000-0000-0000-000000000000

# List all log-analytics solutions in a resource group
Get-AzMonitorLogAnalyticsSolution -ResourceGroupName MyResourceGroup
```

* * *

## <a name="install-a-monitoring-solution"></a>Instalación de una solución de supervisión

### <a name="portal"></a>[Portal](#tab/portal)

Las soluciones de supervisión de Microsoft y asociados están disponibles en [Azure Marketplace](https://azuremarketplace.microsoft.com). Puede buscar las soluciones disponibles e instalarlas mediante el procedimiento siguiente. Al instalar una solución, debe seleccionar un [área de trabajo de Log Analytics](../logs/manage-access.md) donde se instalará la solución y se recopilarán los datos.

1. Desde la [lista de soluciones para la suscripción](#list-installed-monitoring-solutions), haga clic en **Agregar**.
1. Examine o busque una solución. También puede examinar las soluciones desde [este vínculo de búsqueda](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/category/management-tools?page=1&subcategories=management-solutions).
1. Localice la solución de supervisión que quiere y lea su descripción.
1. Haga clic en **Crear** para iniciar el proceso de instalación.
1. Cuando se inicia el proceso de instalación, se le pedirá especificar el área de trabajo de Log Analytics y proporcionar cualquier configuración necesaria para la solución.

![Instalar una solución](media/solutions/install-solution.png)

### <a name="install-a-solution-from-the-community"></a>Instalación de una solución desde la comunidad

Los miembros de la comunidad pueden enviar soluciones de administración a las plantillas de inicio rápido de Azure. Puede instalar estas soluciones directamente o descargar plantillas para hacerlo más adelante.

1. Siga el proceso descrito en [Área de trabajo de Log Analytics y cuenta de Automation](#log-analytics-workspace-and-automation-account) para vincular un área de trabajo y una cuenta.
2. Visite [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/).
3. Busque una solución que le interese.
4. Seleccione la solución en los resultados para ver su información.
5. Haga clic en botón **Deploy to Azure** (Implementar en Azure).
6. Se le pedirá que proporcione información como el grupo de recursos y la ubicación, además de los valores de los parámetros de la solución.
7. Haga clic en **Adquirir** para instalar la solución.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

### <a name="prepare-your-environment"></a>Preparación del entorno

1. Instalación de la CLI de Azure

   Debe [instalar la CLI de Azure](/cli/azure/install-azure-cli) antes de ejecutar los comandos de la referencia de la CLI.  Si lo prefiere, también puede usar Azure Cloud Shell para completar los pasos de este artículo.  Azure Cloud Shell es un entorno de shell interactivo que se utiliza mediante el explorador.  Inicie Cloud Shell con uno de estos métodos:

   - Vaya a [https://shell.azure.com](https://shell.azure.com) para abrir Cloud Shell.

   - Seleccione el botón **Cloud Shell** en la barra de menús de la esquina superior derecha de [Azure Portal](https://portal.azure.com).

1. Inicie sesión.

   Si está usando una instalación local de la CLI, inicie sesión con el comando [az login](/cli/azure/reference-index#az_login).  Siga los pasos que se muestran en el terminal para completar el proceso de autenticación.

    ```azurecli
    az login
    ```

1. Instale la extensión `log-analytics-solution`

   El comando `log-analytics-solution` es una extensión experimental de la CLI de Azure principal. Obtenga más información sobre las referencias de extensión en [Uso de extensiones con la CLI de Azure](/cli/azure/azure-cli-extensions-overview?).

   ```azurecli
   az extension add --name log-analytics-solution
   ```

   Se espera la advertencia siguiente.

   ```output
   The installed extension `log-analytics-solution` is experimental and not covered by customer support.  Please use with discretion.
   ```

### <a name="install-a-solution-with-the-azure-cli"></a>Instalación de una solución con la CLI de Azure

Al instalar una solución, debe seleccionar un [área de trabajo de Log Analytics](../logs/manage-access.md) donde se instalará la solución y se recopilarán los datos.  Con la CLI de Azure, las áreas de trabajo se administran mediante los comandos de referencia [az monitor log-analytics workspace](/cli/azure/monitor/log-analytics/workspace).  Siga el proceso descrito en [Área de trabajo de Log Analytics y cuenta de Automation](#log-analytics-workspace-and-automation-account) para vincular un área de trabajo y una cuenta.

Use [az monitor log-analytics solution create](/cli/azure/monitor/log-analytics/solution) para instalar una solución de supervisión.  Los parámetros entre corchetes son opcionales.

```azurecli
az monitor log-analytics solution create --name
                                         --plan-product
                                         --plan-publisher
                                         --resource-group
                                         --workspace
                                         [--no-wait]
                                         [--tags]
```

Este es un ejemplo de código que crea una solución de Log Analytics para el producto del plan OMSGallery/Containers.

```azurecli
az monitor log-analytics solution create --resource-group MyResourceGroup \
                                         --name Containers({SolutionName}) \
                                         --tags key=value \
                                         --plan-publisher Microsoft  \
                                         --plan-product "OMSGallery/Containers" \
                                         --workspace "/subscriptions/{SubID}/resourceGroups/{ResourceGroup}/providers/ \
                                           Microsoft.OperationalInsights/workspaces/{WorkspaceName}"
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

### <a name="prepare-your-environment"></a>Preparación del entorno

1. Instalar Azure Powershell

   Debe [instalar Azure PowerShell](/powershell/azure/install-az-ps) antes de ejecutar los comandos de referencia de Azure PowerShell. Si lo prefiere, también puede usar Azure Cloud Shell para completar los pasos de este artículo. Azure Cloud Shell es un entorno de shell interactivo que se utiliza mediante el explorador. Inicie Cloud Shell con uno de estos métodos:

   - Vaya a [https://shell.azure.com](https://shell.azure.com) para abrir Cloud Shell.

   - Seleccione el botón **Cloud Shell** en la barra de menús de la esquina superior derecha de [Azure Portal](https://portal.azure.com).

   > [!IMPORTANT]
   > Si bien el módulo **Az.MonitoringSolutions** de PowerShell está en versión preliminar, se debe instalar por separado mediante el cmdlet `Install-Module`. Una vez que este módulo de PowerShell esté disponible con carácter general, formará parte de las futuras versiones del módulo Az de PowerShell y estará disponible de forma predeterminada en Azure Cloud Shell.

   ```azurepowershell-interactive
   Install-Module -Name Az.MonitoringSolutions
   ```

1. Inicie sesión.

   Si usa una instalación local de PowerShell, inicie sesión con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Siga los pasos que se muestran en PowerShell para completar el proceso de autenticación.

   ```azurepowershell
   Connect-AzAccount
   ```

### <a name="install-a-solution-with-azure-powershell"></a>Instalación de una solución con Azure PowerShell

Al instalar una solución, debe seleccionar un [área de trabajo de Log Analytics](../logs/manage-access.md) donde se instalará la solución y se recopilarán los datos. Con Azure PowerShell, las áreas de trabajo se administran con los cmdlets del módulo [Az.MonitoringSolutions](/powershell/module/az.monitoringsolutions) de PowerShell. Siga el proceso descrito en [Área de trabajo de Log Analytics y cuenta de Automation](#log-analytics-workspace-and-automation-account) para vincular un área de trabajo y una cuenta.

Use el cmdlet [New-AzMonitorLogAnalyticsSolution](/powershell/module/az.monitoringsolutions/new-azmonitorloganalyticssolution) para instalar una solución de supervisión. Los parámetros entre corchetes son opcionales.

```azurepowershell
New-AzMonitorLogAnalyticsSolution -ResourceGroupName <string> -Type <string> -Location <string>
-WorkspaceResourceId <string> [-SubscriptionId <string>] [-Tag <hashtable>]
[-DefaultProfile <psobject>] [-Break] [-HttpPipelineAppend <SendAsyncStep[]>]
[-HttpPipelinePrepend <SendAsyncStep[]>] [-Proxy <uri>] [-ProxyCredential <pscredential>]
[-ProxyUseDefaultCredentials] [-WhatIf] [-Confirm] [<CommonParameters>]
```

En el ejemplo siguiente se crea una solución de supervisión de Log Analytics para el área de trabajo de Log Analytics.

```azurepowershell-interactive
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName MyResourceGroup -Name WorkspaceName
New-AzMonitorLogAnalyticsSolution -Type Containers -ResourceGroupName MyResourceGroup -Location $workspace.Location -WorkspaceResourceId $workspace.ResourceId
```

* * *

## <a name="log-analytics-workspace-and-automation-account"></a>Área de trabajo de Log Analytics y cuenta de Automation

Todas las soluciones de supervisión requieren un [área de trabajo de Log Analytics](../logs/manage-access.md) para almacenar los datos recopilados por la solución y para hospedar sus vistas y búsquedas de registros. Algunas soluciones también requieren una [cuenta de Automation](../../automation/automation-security-overview.md) que contenga runbooks y recursos relacionados. El área de trabajo y la cuenta deben cumplir los siguientes requisitos.

* Cada instalación de la solución solo puede utilizar un área de trabajo de Log Analytics y una cuenta de Automation. Puede instalar la solución por separado en varias áreas de trabajo.
* Si una solución requiere una cuenta de Automation, el área de trabajo de Log Analytics y la cuenta de Automation solución deben estar vinculadas. Un área de trabajo de Log Analytics solo puede estar vinculada a una cuenta de Automation y viceversa.

Al instalar una solución mediante Azure Marketplace, se le piden un área de trabajo y una cuenta de Automation. Se creará un vínculo entre ellas, si no están ya vinculadas.

### <a name="verify-the-link-between-a-log-analytics-workspace-and-automation-account"></a>Comprobación de un vínculo entre un área de trabajo de Log Analytics y una cuenta de Automation

Puede comprobar el vínculo entre un área de trabajo de Log Analytics y una cuenta de Automation mediante el procedimiento siguiente.

1. Seleccione la cuenta de Automation en Azure Portal.
1. Vaya a la sección **Related Resources** (Recursos relacionados) del menú y seleccione **Linked workspace** (Área de trabajo vinculada).
1. Si el **área de trabajo** está vinculada a una cuenta de Automation, en esta página se muestra el área de trabajo a la que está vinculada. Si selecciona el nombre del área de trabajo que aparece, se le redirigirá a la página de información general de dicha área de trabajo.

## <a name="remove-a-monitoring-solution"></a>Eliminación de una solución de supervisión

### <a name="portal"></a>[Portal](#tab/portal)

Para quitar una solución instalada mediante el portal, busque en la [lista de soluciones instaladas](#list-installed-monitoring-solutions). Haga clic en el nombre de la solución para abrir su página de resumen y haga clic en **Eliminar**.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para quitar una solución instalada mediante la CLI de Azure, use el comando [az monitor log-analytics solution delete](/cli/azure/monitor/log-analytics/solution#az_monitor_log_analytics_solution_delete).

```azurecli
az monitor log-analytics solution delete --name
                                         --resource-group
                                         [--no-wait]
                                         [--yes]
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Para quitar una solución instalada con Azure PowerShell, use el cmdlet [Remove-AzMonitorLogAnalyticsSolution](/powershell/module/az.monitoringsolutions/remove-azmonitorloganalyticssolution).

```azurepowershell-interactive
Remove-AzMonitorLogAnalyticsSolution  -ResourceGroupName MyResourceGroup -Name WorkspaceName
```

* * *

## <a name="next-steps"></a>Pasos siguientes

* Obtenga un [lista de soluciones de supervisión de Microsoft](../monitor-reference.md).
* Aprenda a [crear consultas](../logs/log-query-overview.md) para analizar los datos recopilados por las soluciones de supervisión.
* Consulte todos los [comandos de la CLI de Azure para Azure Monitor](/cli/azure/azure-cli-reference-for-monitor).