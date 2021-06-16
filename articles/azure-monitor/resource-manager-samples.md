---
title: Ejemplos de plantillas de Resource Manager para Azure Monitor
description: Implementación y configuración de características de Azure Monitor mediante plantillas de Resource Manager
author: bwren
ms.author: bwren
services: azure-monitor
ms.topic: sample
ms.date: 05/18/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b67a1dcb2f6df42c95bb0b5c20ae4397582112db
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111966928"
---
# <a name="resource-manager-template-samples-for-azure-monitor"></a>Ejemplos de plantillas de Resource Manager para Azure Monitor

Azure Monitor se puede implementar y configurar a gran escala mediante una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/syntax.md). Los artículos siguientes ofrecen plantillas de ejemplo para diferentes características de Azure Monitor. Estos ejemplos se pueden modificar con arreglo a sus requisitos y se pueden aplicar utilizando cualquier método estándar para implementar plantillas de Resource Manager. 

## <a name="deploying-the-sample-templates"></a>Implementación de plantillas de ejemplo
A continuación, se incluyen los pasos básicos para utilizar las plantillas de ejemplo:

1. Copie la plantilla y guárdela como un archivo JSON.
2. Modifique los parámetros de su entorno y guárdelos como un archivo JSON.
4. Implemente la plantilla mediante [cualquier método de implementación de plantillas de Resource Manager](../azure-resource-manager/templates/deploy-powershell.md). 

Por ejemplo, use los siguientes comandos para implementar el archivo de plantilla y parámetros en un grupo de recursos mediante PowerShell o la CLI de Azure.


```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName my-subscription
New-AzResourceGroupDeployment -Name AzureMonitorDeployment -ResourceGroupName my-resource-group -TemplateFile azure-monitor-deploy.json -TemplateParameterFile azure-monitor-deploy.parameters.json
```

```azurecli
az login
az deployment group create \
    --name AzureMonitorDeployment \
    --resource-group ResourceGroupofTargetResource \
    --template-file azure-monitor-deploy.json \
    --parameters azure-monitor-deploy.parameters.json
```

## <a name="list-of-sample-templates"></a>Lista de plantillas de ejemplo

- [Agentes](agents/resource-manager-agent.md): implemente y configure la extensión de diagnóstico y el agente de Log Analytics.
- Alertas
  - [Reglas de alertas de registro](alerts/resource-manager-alerts-log.md): alertas de las consultas de registros y del registro de actividad de Azure.
  - [Reglas de alertas de métricas](alerts/resource-manager-alerts-metric.md): alertas de métricas que usan diferentes tipos de lógica.
- [Application Insights](app/resource-manager-app-resource.md)
- [Configuración de diagnóstico](essentials/resource-manager-diagnostic-settings.md): cree una configuración de diagnóstico para reenviar los registros y las métricas desde diferentes tipos de recursos.
- [Consultas de registros](logs/resource-manager-log-queries.md): cree consultas de registros guardadas, en un área de trabajo de Log Analytics.
- [Área de trabajo de Log Analytics](logs/resource-manager-workspace.md): cree el área de trabajo Log Analytics y configure la recopilación de diferentes orígenes de datos a partir del agente de Log Analytics.
- [Libros](visualize/resource-manager-workbooks.md): cree libros.
- [Conclusiones de contenedor](containers/resource-manager-container-insights.md): incorpore clústeres a las conclusiones de contenedor.
- [Azure Monitor para VM](vm/resource-manager-vminsights.md): incorpore máquinas virtuales a Azure Monitor para VM.



## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre las [plantillas de Resource Manager](../azure-resource-manager/templates/overview.md).