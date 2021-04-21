---
title: 'Inicio rápido: Configuración de registros de flujo de grupos de seguridad de red mediante una plantilla de Azure Resource Manager (plantilla de ARM)'
description: Aprenda cómo habilitar los registros de flujo de los grupos de seguridad de red (NSG) mediante programación con una plantilla de Azure Resource Manager y Azure PowerShell.
services: network-watcher
author: damendo
ms.author: damendo
ms.date: 01/07/2021
ms.topic: quickstart
ms.service: network-watcher
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: df0ccb5bf5ecd60d80526085983e35abf58e9966
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107532432"
---
# <a name="quickstart-configure-network-security-group-flow-logs-by-using-an-arm-template"></a>Inicio rápido: Configuración de registros de flujo de grupos de seguridad de red mediante una plantilla de Resource Manager

En este inicio rápido, aprenderá cómo habilitar los [registros de flujo de los grupos de seguridad de red (NSG)](network-watcher-nsg-flow-logging-overview.md) con una plantilla de [Azure Resource Manager](../azure-resource-manager/management/overview.md) y Azure PowerShell.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Comenzaremos con una información general de las propiedades del objeto de registro de flujo del grupo de seguridad de red. Se proporcionan plantillas de ejemplo. A continuación, usamos una instancia local de Azure PowerShell para implementar la plantilla.

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abre en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-networkwatcher-flowLogs-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-networkwatcher-flowlogs-create).

:::code language="json" source="~/quickstart-templates/101-networkwatcher-flowlogs-create/azuredeploy.json":::

Estos recursos se definen en la plantilla:

- [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
- [Microsoft.Resources/deployments](/azure/templates/microsoft.resources/deployments)

## <a name="nsg-flow-logs-object"></a>Objeto de los registros de flujo de NSG

En el código siguiente se muestra un objeto de registros de flujo del grupo de seguridad de red y sus parámetros. Para crear un recurso `Microsoft.Network/networkWatchers/flowLogs`, agregue este código a la sección de recursos de la plantilla.

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

Para obtener información general completa de las propiedades del objeto de registros del flujo del grupo de seguridad de red, consulte [Microsoft.Network networkWatchers/flowLogs](/azure/templates/microsoft.network/networkwatchers/flowlogs).

## <a name="create-your-template"></a>Creación de la plantilla

Si es la primera vez que utiliza plantillas de Resource Manager, consulte los artículos siguientes para más información sobre dichas plantillas:

- [Implementación de recursos con las plantillas de ARM y Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md#deploy-local-template-or-bicep-file)
- [Tutorial: Creación e implementación de su primera plantilla de Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)

El ejemplo siguiente es una plantilla completa. También es la versión más simple de la plantilla. El ejemplo contiene los parámetros mínimos que se pasan para configurar los registros de flujo del grupo de seguridad de red. Para más ejemplos, consulte el artículo de información general [Configuración de registros de flujo de NSG a partir de una plantilla de Azure Resource Manager](network-watcher-nsg-flow-logging-azure-resource-manager.md).

### <a name="example"></a>Ejemplo

La siguiente plantilla habilita los registros de flujo para un grupo de seguridad de red y, a continuación, almacena los registros en una cuenta de almacenamiento específica:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "apiProfile": "2019-09-01",
  "resources": [
    {
      "name": "NetworkWatcher_centraluseuap/Microsoft.NetworkDalanDemoPerimeterNSG",
      "type": "Microsoft.Network/networkWatchers/FlowLogs/",
      "location": "centraluseuap",
      "apiVersion": "2019-09-01",
      "properties": {
        "targetResourceId": "/subscriptions/<subscription Id>/resourceGroups/DalanDemo/providers/Microsoft.Network/networkSecurityGroups/PerimeterNSG",
        "storageId": "/subscriptions/<subscription Id>/resourceGroups/MyCanaryFlowLog/providers/Microsoft.Storage/storageAccounts/storagev2ira",
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
> - El nombre del recurso utiliza el formato _ParentResource_ChildResource_. En este caso, el recurso primario es la instancia regional de Azure Network Watcher:
>    - **Format**: NetworkWatcher_RegionName
>    - **Ejemplo**: NetworkWatcher_centraluseuap
> - `targetResourceId` es el identificador de recurso del NSG de destino.
> - `storageId` es el identificador de recurso de la cuenta de almacenamiento de destino.

## <a name="deploy-the-template"></a>Implementación de la plantilla

En este tutorial se da por supuesto que tiene un grupo de recursos existente y un grupo de seguridad de red en el que puede habilitar el registro de flujo.

Puede guardar cualquiera de las plantillas de ejemplo que se muestran en este artículo localmente como *azuredeploy.json*. Actualice los valores de las propiedades para que apunten a recursos válidos de su suscripción.

Para implementar la plantilla, ejecute el siguiente comando en Azure PowerShell:

```azurepowershell-interactive
$context = Get-AzSubscription -SubscriptionId <subscription Id>
Set-AzContext $context
New-AzResourceGroupDeployment -Name EnableFlowLog -ResourceGroupName NetworkWatcherRG `
    -TemplateFile "C:\MyTemplates\azuredeploy.json"
```

> [!NOTE]
> Estos comandos implementan un recurso en el grupo de recursos NetworkWatcherRG de ejemplo y no en el grupo de recursos que contiene el grupo de seguridad de red.

## <a name="validate-the-deployment"></a>Validación de la implementación

Tiene dos opciones para ver si la implementación se realizó correctamente:

- La consola de PowerShell muestra `ProvisioningState` como `Succeeded`.
- Vaya a la [página del portal de registros de flujo del grupo de seguridad de red](https://ms.portal.azure.com/#blade/Microsoft_Azure_Network/NetworkWatcherMenuBlade/flowLogs) para confirmar los cambios.

Si se produjeron problemas con la implementación, consulte [Solución de errores comunes de implementación de Azure con Azure Resource Manager](../azure-resource-manager/templates/common-deployment-errors.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede eliminar recursos de Azure mediante el modo de implementación completa. Para eliminar un recurso de registros de flujo, especifique una implementación en modo completo sin incluir el recurso que quiere eliminar. Más información sobre el [modo de implementación completo](../azure-resource-manager/templates/deployment-modes.md#complete-mode).

También puede deshabilitar un registro de flujo del grupo de seguridad de red en Azure Portal:

1. Inicie sesión en Azure Portal.
1. Seleccione **Todos los servicios**. En el cuadro **Filtrar**, escriba **network watcher**. En los resultados de la búsqueda, seleccione **Network Watcher**.
1. En **Registros**, seleccione **Registro de flujo de NSG**.
1. En la lista de grupos de seguridad de red, seleccione aquel para el que desea deshabilitar los registros de flujo.
1. En **Configuración de los registros de flujo**, seleccione **Desactivado**.
1. Seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido a habilitar los registros de flujo del grupo de seguridad de red mediante una plantilla de Resource Manager. A continuación, aprenda a visualizar los datos de flujo del grupo de seguridad de red mediante una de estas opciones:

- [Microsoft Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md)
- [Herramientas de código abierto](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)
- [Análisis de tráfico de Azure](traffic-analytics.md)
