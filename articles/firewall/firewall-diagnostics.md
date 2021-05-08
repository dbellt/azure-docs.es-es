---
title: Supervisión de métricas y registros de Azure Firewall
description: En este artículo aprenderá a habilitar y administrar los registros y las métricas de Azure Firewall.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 11/04/2020
ms.author: victorh
ms.openlocfilehash: a682523b9486886d5be3c96a372fe36c81582649
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107988647"
---
# <a name="monitor-azure-firewall-logs-and-metrics"></a>Supervisión de métricas y registros de Azure Firewall

Puede supervisar Azure Firewall mediante los registros del firewall. También puede usar los registros de actividad para auditar las operaciones de los recursos de Azure Firewall. Con las métricas, puede ver los contadores de rendimiento en el portal.

Se puede acceder a algunos de estos registros mediante el portal. Se pueden enviar registros a los [registros de Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md), a Storage y a Event Hubs, y se pueden analizar en los registros de Azure Monitor o mediante otras herramientas como Excel y Power BI.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar debe leer el artículo acerca de las [métricas y registros de Azure Firewall](logs-and-metrics.md), una introducción a los registros y las métricas de diagnóstico disponibles para Azure Firewall.

## <a name="enable-diagnostic-logging-through-the-azure-portal"></a>Habilitación del registro de diagnóstico mediante Azure Portal

Puede tardar algunos minutos en que los datos aparezcan en los registros después de completar este procedimiento para activar el registro de diagnóstico. Si no ve nada a primera vista, vuelva a comprobarlo en unos minutos.

1. En Azure Portal, abra el grupo de recursos de firewall y seleccione el firewall.
2. En **Supervisión**, seleccione **Configuración de diagnóstico**.

   En Azure Firewall, hay cuatro registros específicos del servicio disponibles:

   * AzureFirewallApplicationRule
   * AzureFirewallNetworkRule
   * AzureFirewallDnsProxy


3. Seleccione **Agregar configuración de diagnóstico**. En la página **Configuración de diagnóstico**, se encuentran las opciones de configuración de los registros de diagnóstico.
5. En este ejemplo, los registros de Azure Monitor almacenan los registros, de modo que escriba **Análisis de registros de firewall** como nombre.
6. En **Registro**, seleccione **AzureFirewallApplicationRule**, **AzureFirewallNetworkRule** y **AzureFirewallDnsProxy** para recopilar los registros.
7. Seleccione **Enviar a Log Analytics** para configurar el área de trabajo.
8. Seleccione su suscripción.
9. Seleccione **Guardar**.

## <a name="enable-diagnostic-logging-by-using-powershell"></a>Habilitación del registro de diagnóstico con PowerShell

El registro de actividades se habilita automáticamente para todos los recursos de Resource Manager. Se debe habilitar el registro de diagnóstico para empezar a recopilar los datos disponibles mediante esos registros.

Para habilitar el registro de diagnóstico con PowerShell, siga estos pasos:

1. Anote el identificador de recurso del área de trabajo de Log Analytics donde se almacenan los datos de registro. Este valor tiene el formato: `/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/microsoft.operationalinsights/workspaces/<workspace name>`.

   Puede usar cualquier área de trabajo de la suscripción. Para buscar esta información, se puede usar Azure Portal. La información se encuentra en la página **Propiedades** del recurso.

2. Observe el identificador de recurso del firewall para el que se ha habilitado el registro. Este valor tiene el formato: `/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/azureFirewalls/<Firewall name>`.

   Para buscar esta información, use Azure Portal.

3. Habilite el registro de diagnóstico para todos los registros y las métricas mediante el siguiente cmdlet de PowerShell:

   ```powershell
   $diagSettings = @{
      Name = 'toLogAnalytics'
      ResourceId = '/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/azureFirewalls/<Firewall name>'
      WorkspaceId = '/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/microsoft.operationalinsights/workspaces/<workspace name>'
      Enabled = $true
   }
   Set-AzDiagnosticSetting  @diagSettings 
   ```

## <a name="enable-diagnostic-logging-by-using-the-azure-cli"></a>Habilitación del registro de diagnóstico con la CLI de Azure

El registro de actividades se habilita automáticamente para todos los recursos de Resource Manager. Se debe habilitar el registro de diagnóstico para empezar a recopilar los datos disponibles mediante esos registros.

Para habilitar el registro de diagnóstico con la CLI de Azure, siga estos pasos:

1. Anote el identificador de recurso del área de trabajo de Log Analytics donde se almacenan los datos de registro. Este valor tiene el formato: `/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/azureFirewalls/<Firewall name>`.

   Puede usar cualquier área de trabajo de la suscripción. Para buscar esta información, se puede usar Azure Portal. La información se encuentra en la página **Propiedades** del recurso.

2. Observe el identificador de recurso del firewall para el que se ha habilitado el registro. Este valor tiene el formato: `/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/azureFirewalls/<Firewall name>`.

   Para buscar esta información, use Azure Portal.

3. Habilite el registro de diagnóstico para todos los registros y las métricas mediante el siguiente comando de la CLI de Azure:

   ```azurecli-interactive
   az monitor diagnostic-settings create -n 'toLogAnalytics'
      --resource '/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/azureFirewalls/<Firewall name>'
      --workspace '/subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/microsoft.operationalinsights/workspaces/<workspace name>'
      --logs '[{\"category\":\"AzureFirewallApplicationRule\",\"Enabled\":true}, {\"category\":\"AzureFirewallNetworkRule\",\"Enabled\":true}, {\"category\":\"AzureFirewallDnsProxy\",\"Enabled\":true}]' 
      --metrics '[{\"category\": \"AllMetrics\",\"enabled\": true}]'
   ```

## <a name="view-and-analyze-the-activity-log"></a>Visualización y análisis del registro de actividades

Puede ver y analizar los datos del registro de actividades con cualquiera de los métodos siguientes:

* **Herramientas de Azure**: permiten recuperar información de los registros de actividades mediante Azure PowerShell, la CLI de Azure, la API REST de Azure o Azure Portal. En el artículo [Operaciones de actividades con Resource Manager](../azure-resource-manager/management/view-activity-logs.md) se detallan instrucciones paso a paso de cada método.
* **Power BI**: si todavía no tiene una cuenta de [Power BI](https://powerbi.microsoft.com/pricing), puede probarlo gratis. Con el [paquete de contenido de los registros de actividades de Azure para Power BI](https://powerbi.microsoft.com/en-us/documentation/powerbi-content-pack-azure-audit-logs/), puede analizar sus datos con los paneles preconfigurados que puede personalizar o usar tal cual.
* **Azure Sentinel**: Los registros de Azure Firewall se pueden conectar a Azure Sentinel, de forma que es posible ver los datos de registro en los libros, usarlos para crear alertas personalizadas e incorporarlos para mejorar su investigación. El conector de datos de Azure Firewall en Azure Sentinel se encuentra actualmente en versión preliminar pública. Para más información, consulte [Conexión de datos desde Azure Firewall](../sentinel/connect-azure-firewall.md).

## <a name="view-and-analyze-the-network-and-application-rule-logs"></a>Ver y analizar los registros de la regla de red y de aplicación

Los [registros de Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md) recopilan los archivos de contador y de registros de eventos. Incluye visualizaciones y eficaces funciones de búsqueda para analizar los registros.

Para ver consultas de ejemplo, consulte [Ejemplos de Log Analytics en Azure Firewall](./firewall-workbook.md).

El [libro de Azure Firewall](firewall-workbook.md) proporciona un lienzo flexible para el análisis de datos de Azure Firewall. Puede utilizarlo para crear informes visuales completos en Azure Portal. Puede acceder a varias instancias de Firewall implementadas en Azure y combinarlas en experiencias interactivas unificadas.

También puede conectarse a la cuenta de almacenamiento y recuperar las entradas del registro de JSON de los registros de acceso y rendimiento. Después de descargar los archivos JSON, se pueden convertir a CSV y consultarlos en Excel, PowerBI o cualquier otra herramienta de visualización de datos.

> [!TIP]
> Si está familiarizado con Visual Studio y con los conceptos básicos de cambio de los valores de constantes y variables de C#, puede usar las [herramientas convertidoras de registros](https://github.com/Azure-Samples/networking-dotnet-log-converter) que encontrará en GitHub.

## <a name="view-metrics"></a>Visualización de métricas
Navegue a una instancia de Azure Firewall y, en **Supervisión** seleccione **Métricas**. Para ver los valores disponibles, seleccione la lista desplegable **MÉTRICA**.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha configurado el firewall para recopilar registros, puede explorar los registros de Azure Monitor para ver los datos.

[Supervisión de registros mediante el libro de Azure Firewall](firewall-workbook.md)

[Soluciones de supervisión de redes en registros de Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md)
