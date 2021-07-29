---
title: 'Network Watcher: creación de registros de flujo de NSG mediante una plantilla de Azure Resource Manager'
description: Use una plantilla de Azure Resource Manager y PowerShell para configurar fácilmente los registros de flujo del grupo de seguridad de red.
services: network-watcher
documentationcenter: na
author: damendo
manager: twooley
editor: ''
tags: azure-resource-manager
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: damendo
ms.custom: fasttrack-edit, devx-track-azurepowershell
ms.openlocfilehash: 6aab2d1d681cbfd55cc275b1f647562d54a08f0c
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110791337"
---
# <a name="configure-nsg-flow-logs-from-an-azure-resource-manager-template"></a>Configuración de registros de flujo de NSG a partir de una plantilla de Azure Resource Manager

> [!div class="op_single_selector"]
> - [Azure Portal](network-watcher-nsg-flow-logging-portal.md)
> - [PowerShell](network-watcher-nsg-flow-logging-powershell.md)
> - [CLI de Azure](network-watcher-nsg-flow-logging-cli.md)
> - [REST API](network-watcher-nsg-flow-logging-rest.md)
> - [Azure Resource Manager](network-watcher-nsg-flow-logging-azure-resource-manager.md)


[Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/) es la forma nativa y eficaz de Azure de administrar su [infraestructura como código](/azure/devops/learn/what-is-infrastructure-as-code).

En este artículo se muestra cómo habilitar los [registros de flujo de NSG](./network-watcher-nsg-flow-logging-overview.md) mediante programación con una plantilla de Azure Resource Manager y Azure PowerShell. Comenzaremos con una visión general de las propiedades del objeto del registro de flujo de NSG, seguida de algunas plantillas de ejemplo. A continuación, usaremos una instancia local de PowerShell para implementar la plantilla.


## <a name="nsg-flow-logs-object"></a>Objeto de los registros de flujo de NSG

A continuación se muestra el objeto de los registros de flujo de NSG con todos los parámetros.
Para obtener una introducción completa de las propiedades, lea la [referencia de la plantilla de registros de flujo de NSG](/azure/templates/microsoft.network/2019-11-01/networkwatchers/flowlogs#RetentionPolicyParameters).

```json
{
  "name": "string",
  "type": "Microsoft.Network/networkWatchers/flowLogs",
  "location": "string",
  "apiVersion": "2019-09-01",
  "properties": {
    "targetResourceId": "string",
    "storageId": "string",
    "enabled": "boolean",
    "flowAnalyticsConfiguration": {
      "networkWatcherFlowAnalyticsConfiguration": {
         "enabled": "boolean",
         "workspaceResourceId": "string",
          "trafficAnalyticsInterval": "integer"
        },
        "retentionPolicy": {
           "days": "integer",
           "enabled": "boolean"
         },
        "format": {
           "type": "string",
           "version": "integer"
         }
      }
    }
  }
```
Para crear un recurso Microsoft.Network/networkWatchers/flowLogs, agregue el JSON anterior a la sección resources de la plantilla.


## <a name="creating-your-template"></a>Creación de la plantilla

Si es la primera vez que usa plantillas de Azure Resource Manager, puede obtener más información sobre ellas con los vínculos siguientes.

* [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md#deploy-local-template)
* [Tutorial: Creación e implementación de la primera plantilla de Azure Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md?tabs=azure-powershell)


A continuación se muestran dos ejemplos de plantillas completas para configurar los registros de flujo de NSG.

**Ejemplo 1**:  La versión más simple del código anterior con un número mínimo de parámetros pasados. La siguiente plantilla habilita los registros de flujo de NSG en un NSG de destino y los almacena en una cuenta de almacenamiento específica.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "apiProfile": "2019-09-01",
  "resources": [
 {
    "name": "NetworkWatcher_centraluseuap/Microsoft.NetworkDalanDemoPerimeterNSG",
    "type": "Microsoft.Network/networkWatchers/FlowLogs/",
    "location": "centraluseuap",
    "apiVersion": "2019-09-01",
    "properties": {
      "targetResourceId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/DalanDemo/providers/Microsoft.Network/networkSecurityGroups/PerimeterNSG",
      "storageId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/MyCanaryFlowLog/providers/Microsoft.Storage/storageAccounts/storagev2ira",
      "enabled": true,
      "flowAnalyticsConfiguration": {},
      "retentionPolicy": {},
      "format": {}
    }

  }
  ]
}
```

> [!NOTE]
> * El nombre del recurso tiene el formato "Parent Resource_Child resource". En este caso, el recurso primario es la instancia regional de Network Watcher (formato: NetworkWatcher_NombreDeRegión. Ejemplo: NetworkWatcher_centraluseuap)
> * targetResourceId es el id. de recurso del NSG de destino
> * storageId es el id. de recurso de la cuenta de almacenamiento de destino

**Ejemplo 2**: Las siguientes plantillas habilitan los registros de flujo de NSG (versión 2) con una retención de 5 días. Además, se habilita el Análisis de tráfico con un intervalo de procesamiento de 10 minutos.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "apiProfile": "2019-09-01",
  "resources": [
    {
      "name": "NetworkWatcher_centraluseuap/Microsoft.NetworkDalanDemoPerimeterNSG",
      "type": "Microsoft.Network/networkWatchers/FlowLogs/",
      "location": "centraluseuap",
      "apiVersion": "2019-09-01",
      "properties": {
        "targetResourceId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/DalanDemo/providers/Microsoft.Network/networkSecurityGroups/PerimeterNSG",
        "storageId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/MyCanaryFlowLog/providers/Microsoft.Storage/storageAccounts/storagev2ira",
        "enabled": true,
        "flowAnalyticsConfiguration": {
          "networkWatcherFlowAnalyticsConfiguration": {
            "enabled": true,
            "workspaceResourceId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/defaultresourcegroup-wcus/providers/Microsoft.OperationalInsights/workspaces/1c4f42e5-3a02-4146-ac9b-3051d8501db0",
            "trafficAnalyticsInterval": 10
          }
        },
        "retentionPolicy": {
          "days": 5,
          "enabled": true
        },
        "format": {
          "type": "JSON",
          "version": 2
        }
      }
    }
  ]
}
```

## <a name="deploying-your-azure-resource-manager-template"></a>Implementación de la plantilla de Azure Resource Manager

En este tutorial se da por supuesto que tiene un grupo de recursos existente y un NSG en el que puede habilitar el registro de flujo.
Puede guardar cualquiera de las plantillas de ejemplo anteriores de forma local como `azuredeploy.json`. Actualice los valores de las propiedades para que apunten a recursos válidos de su suscripción.

Para implementar la plantilla, ejecute el siguiente comando en PowerShell.
```azurepowershell
$context = Get-AzSubscription -SubscriptionId <SubscriptionId>
Set-AzContext $context
New-AzResourceGroupDeployment -Name EnableFlowLog -ResourceGroupName NetworkWatcherRG `
    -TemplateFile "C:\MyTemplates\azuredeploy.json"
```

> [!NOTE]
> Los comandos anteriores implementan un recurso en el grupo de recursos NetworkWatcherRG y no en el grupo de recursos que contiene NSG.


## <a name="verifying-your-deployment"></a>Comprobación de la implementación

Hay un par de maneras de comprobar si la implementación se ha realizado correctamente. La consola de PowerShell debería mostrar "ProvisioningState" como "Succeeded". Además, puede visitar la [página del portal de registros de flujo de NSG](https://ms.portal.azure.com/#blade/Microsoft_Azure_Network/NetworkWatcherMenuBlade/flowLogs) para confirmar los cambios. Si se produjeron problemas con la implementación, eche un vistazo a [Solución de errores comunes de implementación de Azure con Azure Resource Manager](../azure-resource-manager/templates/common-deployment-errors.md).

## <a name="deleting-your-resource"></a>Eliminación del recurso
Azure permite la eliminación de recursos mediante el modo de implementación "Completo". Para eliminar un recurso de registros de flujo, especifique una implementación en modo completo sin incluir el recurso que quiere eliminar. Obtenga más información sobre el [modo de implementación completo](../azure-resource-manager/templates/deployment-modes.md#complete-mode).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo visualizar los registros de flujo de NSG con:
* [Microsoft Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md)
* [Herramientas de código abierto](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)
* [Análisis de tráfico de Azure](./traffic-analytics.md)
