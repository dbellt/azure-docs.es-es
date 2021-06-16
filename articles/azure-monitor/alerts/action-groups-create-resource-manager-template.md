---
title: Creación de grupos de acciones con plantillas de Resource Manager
description: Aprenda a crear un grupo de acciones mediante una plantilla de Azure Resource Manager.
author: dkamstra
services: azure-monitor
ms.topic: conceptual
ms.date: 02/19/2021
ms.author: dukek
ms.openlocfilehash: fe28498f677c2025cea9da329a3b029cda04a199
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111949378"
---
# <a name="create-an-action-group-with-a-resource-manager-template"></a>Creación de un grupo de acciones con una plantilla de Resource Manager
En este artículo se muestra cómo utilizar una [plantilla de Azure Resource Manager](../../azure-resource-manager/templates/syntax.md) para configurar grupos de acciones. Mediante el uso de plantillas, puede configurar automáticamente grupos de acciones que se pueden reutilizar en determinados tipos de alertas. Estos grupos de acciones garantizan que se notifique a todas las entidades correctas cuando se desencadene una alerta.

Los pasos básicos son:

1. Crear una plantilla en forma de archivo JSON que describa cómo crear la alerta del grupo de acciones.

2. Implemente la plantilla mediante [cualquier método de implementación](../../azure-resource-manager/templates/deploy-powershell.md).

## <a name="resource-manager-templates-for-an-action-group"></a>Plantillas de Resource Manager para un grupo de acciones

Para crear un grupo de acciones mediante una plantilla de Resource Manager, cree un recurso del tipo `Microsoft.Insights/actionGroups`. A continuación, rellene todas las propiedades relacionadas. Aquí hay dos plantillas de ejemplo que crean un grupo de acciones.

En la primera plantilla se describe cómo crear una plantilla de Resource Manager para un grupo de acciones, en el que las definiciones de acción están codificadas en la plantilla. En la segunda plantilla, se describe cómo crear una plantilla que toma la información de configuración de webhook como parámetros de entrada cuando se implementa la plantilla.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "actionGroupName": {
      "type": "string",
      "metadata": {
        "description": "Unique name (within the Resource Group) for the Action group."
      }
    },
    "actionGroupShortName": {
      "type": "string",
      "metadata": {
        "description": "Short name (maximum 12 characters) for the Action group."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Insights/actionGroups",
      "apiVersion": "2019-03-01",
      "name": "[parameters('actionGroupName')]",
      "location": "Global",
      "properties": {
        "groupShortName": "[parameters('actionGroupShortName')]",
        "enabled": true,
        "smsReceivers": [
          {
            "name": "contosoSMS",
            "countryCode": "1",
            "phoneNumber": "5555551212"
          },
          {
            "name": "contosoSMS2",
            "countryCode": "1",
            "phoneNumber": "5555552121"
          }
        ],
        "emailReceivers": [
          {
            "name": "contosoEmail",
            "emailAddress": "devops@contoso.com",
            "useCommonAlertSchema": true

          },
          {
            "name": "contosoEmail2",
            "emailAddress": "devops2@contoso.com",
            "useCommonAlertSchema": true
          }
        ],
        "webhookReceivers": [
          {
            "name": "contosoHook",
            "serviceUri": "http://requestb.in/1bq62iu1",
            "useCommonAlertSchema": true
          },
          {
            "name": "contosoHook2",
            "serviceUri": "http://requestb.in/1bq62iu2",
            "useCommonAlertSchema": true
          }
        ]
      }
    }
  ],
  "outputs":{
      "actionGroupId":{
          "type":"string",
          "value":"[resourceId('Microsoft.Insights/actionGroups',parameters('actionGroupName'))]"
      }
  }
}
```

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "actionGroupName": {
      "type": "string",
      "metadata": {
        "description": "Unique name (within the Resource Group) for the Action group."
      }
    },
    "actionGroupShortName": {
      "type": "string",
      "metadata": {
        "description": "Short name (maximum 12 characters) for the Action group."
      }
    },
    "webhookReceiverName": {
      "type": "string",
      "metadata": {
        "description": "Webhook receiver service Name."
      }
    },    
    "webhookServiceUri": {
      "type": "string",
      "metadata": {
        "description": "Webhook receiver service URI."
      }
    }    
  },
  "resources": [
    {
      "type": "Microsoft.Insights/actionGroups",
      "apiVersion": "2019-03-01",
      "name": "[parameters('actionGroupName')]",
      "location": "Global",
      "properties": {
        "groupShortName": "[parameters('actionGroupShortName')]",
        "enabled": true,
        "smsReceivers": [
        ],
        "emailReceivers": [
        ],
        "webhookReceivers": [
          {
            "name": "[parameters('webhookReceiverName')]",
            "serviceUri": "[parameters('webhookServiceUri')]",
            "useCommonAlertSchema": true
          }
        ]
      }
    }
  ],
  "outputs":{
      "actionGroupResourceId":{
          "type":"string",
          "value":"[resourceId('Microsoft.Insights/actionGroups',parameters('actionGroupName'))]"
      }
  }
}
```


## <a name="next-steps"></a>Pasos siguientes
* Más información sobre los [grupos de acciones](./action-groups.md).
* Obtenga más información sobre [alertas](./alerts-overview.md).
* Aprenda a agregar [alertas mediante una plantilla de Resource Manager](./alerts-activity-log.md).