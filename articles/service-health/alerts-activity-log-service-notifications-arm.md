---
title: Recepción de alertas de registro de actividad en las notificaciones del servicio de Azure mediante una plantilla de Resource Manager
description: Reciba notificaciones por SMS, correo electrónico o webhook cuando se produzcan eventos en el servicio de Azure.
ms.date: 06/29/2020
ms.topic: quickstart
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 730c023de61275d95fe594642149770af42b34b9
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107535751"
---
# <a name="quickstart-create-activity-log-alerts-on-service-notifications-using-an-arm-template"></a>Inicio rápido: Creación de alertas del registro de actividad en notificaciones del servicio mediante una plantilla de Resource Manager

En este artículo se explica cómo configurar las alertas del registro de actividad para las notificaciones de mantenimiento de un servicio mediante una plantilla de Resource Manager.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Las notificaciones de mantenimiento del servicio se almacenan en el [registro de actividad de Azure](../azure-monitor/essentials/platform-logs-overview.md). Debido al volumen posiblemente grande de la información almacenada en el registro de actividad, hay una interfaz de usuario independiente que facilita la visualización y la configuración de alertas en las notificaciones de mantenimiento del servicio.

Puede recibir una alerta cuando Azure envía notificaciones de estado del servicio a la suscripción de Azure. Puede configurar la alerta en función de:

- La clase de notificación de estado del servicio (problemas de servicio, mantenimiento planificado y avisos de estado).
- La suscripción afectada.
- Los servicios afectados.
- Las regiones afectadas.

> [!NOTE]
> Las notificaciones de estado del servicio no envían una alerta relativa a los eventos de estado de recursos.

También puede configurar a quién se debe enviar la alerta:

- Seleccione un grupo de acciones existente.
- Cree un nuevo grupo de acciones (que puede usarse para futuras alertas).

Para más información sobre los grupos de acciones, consulte [Creación y administración de grupos de acciones](../azure-monitor/alerts/action-groups.md).

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Para ejecutar los comandos desde la máquina local, instale la CLI de Azure o los módulos de Azure PowerShell. Para más información, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli) e [Instalar Azure Powershell](/powershell/azure/install-az-ps).

## <a name="review-the-template"></a>Revisión de la plantilla

En el siguiente ejemplo se crea un grupo de acciones con un destino de correo electrónico y se habilitan todas las notificaciones de estado de servicio de la suscripción de destino. Guarde esta plantilla como *CreateServiceHealthAlert.json*.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "actionGroups_name": {
      "type": "String",
      "defaultValue": "SubHealth"
    },
    "activityLogAlerts_name": {
      "type": "String",
      "defaultValue": "ServiceHealthActivityLogAlert"
    },
    "emailAddress": {
      "type": "string"
    }
  },
  "variables": {
    "alertScope": "[concat('/','subscriptions','/',subscription().subscriptionId)]"
  },
  "resources": [
    {
      "comments": "Action Group",
      "type": "microsoft.insights/actionGroups",
      "apiVersion": "2019-06-01",
      "name": "[parameters('actionGroups_name')]",
      "location": "Global",
      "scale": null,
      "dependsOn": [],
      "tags": {},
      "properties": {
        "groupShortName": "[parameters('actionGroups_name')]",
        "enabled": true,
        "emailReceivers": [
          {
            "name": "[parameters('actionGroups_name')]",
            "emailAddress": "[parameters('emailAddress')]"
          }
        ],
        "smsReceivers": [],
        "webhookReceivers": []
      }
    },
    {
      "comments": "Service Health Activity Log Alert",
      "type": "microsoft.insights/activityLogAlerts",
      "apiVersion": "2017-04-01",
      "name": "[parameters('activityLogAlerts_name')]",
      "location": "Global",
      "scale": null,
      "dependsOn": [
        "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]"
      ],
      "tags": {},
      "properties": {
        "scopes": [
          "[variables('alertScope')]"
        ],
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "ServiceHealth"
            },
            {
              "field": "properties.incidentType",
              "equals": "Incident"
            }
          ]
        },
        "actions": {
          "actionGroups": [
            {
              "actionGroupId": "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]",
              "webhookProperties": {}
            }
          ]
        },
        "enabled": true,
        "description": ""
      }
    }
  ]
}
```

La plantilla define dos recursos:

- [Microsoft.Insights/actionGroups](/azure/templates/microsoft.insights/actiongroups)
- [Microsoft.Insights/activityLogAlerts](/azure/templates/microsoft.insights/activityLogAlerts)

## <a name="deploy-the-template"></a>Implementación de la plantilla

Implemente la plantilla mediante cualquier método estándar de [implementación de una plantilla de Resource Manager](../azure-resource-manager/templates/deploy-portal.md) como en los ejemplos siguientes con la CLI y PowerShell. Reemplace los valores de ejemplo del **grupo de recursos** y **dirección de correo electrónico** por los valores adecuados para su entorno.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az login
az deployment group create --name CreateServiceHealthAlert --resource-group my-resource-group --template-file CreateServiceHealthAlert.json --parameters emailAddress='user@contoso.com'
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName my-subscription
New-AzResourceGroupDeployment -Name CreateServiceHealthAlert -ResourceGroupName my-resource-group -TemplateFile CreateServiceHealthAlert.json -emailAddress user@contoso.com
```

---

## <a name="validate-the-deployment"></a>Validación de la implementación

Para comprobar que se ha creado el área de trabajo, utilice uno de los comandos siguientes. Reemplace los valores de ejemplo del **grupo de recursos** por los valores que usó anteriormente.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az monitor activity-log alert show --resource-group my-resource-group --name ServiceHealthActivityLogAlert
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Get-AzActivityLogAlert -ResourceGroupName my-resource-group -Name ServiceHealthActivityLogAlert
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Si planea seguir trabajando en otros inicios rápidos y tutoriales, considere la posibilidad de dejar estos recursos activos. Cuando ya no lo necesite, elimine el grupo de recursos; de este modo, se eliminarán también la regla de alertas y los recursos relacionados. Para eliminar el grupo de recursos mediante la CLI de Azure o Azure PowerShell

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az group delete --name my-resource-group
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Remove-AzResourceGroup -Name my-resource-group
```

---

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre los [procedimientos recomendados para la configuración de alertas de Azure Service Health](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUa).
- Obtenga información sobre cómo [configurar notificaciones de inserción móviles de Azure Service Health](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUw).
- Obtenga información acerca de cómo [configurar notificaciones de webhook para los sistemas de administración de problemas existentes](service-health-alert-webhook-guide.md).
- Más información acerca de las [Notificaciones del estado del servicio](service-notifications.md).
- Más información sobre la [Limitación del número de notificaciones](../azure-monitor/alerts/alerts-rate-limiting.md).
- Revise el [Esquema de webhook de alertas del registro de actividad](../azure-monitor/alerts/activity-log-alerts-webhook.md).
- Consulte la [introducción a las alertas del registro de actividad](../azure-monitor/alerts/alerts-overview.md) y aprenda cómo puede recibir alertas.
- Más información sobre los [grupos de acciones](../azure-monitor/alerts/action-groups.md).
