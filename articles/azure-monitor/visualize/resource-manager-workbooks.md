---
title: Ejemplos de plantillas de Azure Resource Manager para consultas de libros
description: Plantillas de ejemplo de Azure Resource Manager para implementar libros de Azure Monitor.
ms.topic: sample
author: bwren
ms.author: bwren
ms.date: 05/18/2020
ms.openlocfilehash: b9f8907b8bfe4cac7ddb7f3daeda45a56cedea0a
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958167"
---
# <a name="resource-manager-template-samples-for-workbooks-in-azure-monitor"></a>Ejemplos de plantillas de Azure Resource Manager para libros en Azure Monitor
Este artículo incluye ejemplos de [plantillas de Azure Resource Manager](../../azure-resource-manager/templates/syntax.md) para crear libros en Azure Monitor. Cada ejemplo incluye un archivo de plantilla y un archivo de parámetros con valores de ejemplo para la plantilla.

[!INCLUDE [azure-monitor-samples](../../../includes/azure-monitor-resource-manager-samples.md)]

Los libros pueden ser complejos, por lo que una estrategia típica es crear el libro en Azure Portal y, a continuación, generar una plantilla de Resource Manager. Consulte los detalles de este método en [Plantilla de Azure Resource Manager para implementar libros](../visualize/workbooks-automate.md).

## <a name="create-a-workbook"></a>Creación de un libro
En el ejemplo siguiente se crean cuatro libros sencillos.


### <a name="template-file"></a>Archivo de plantilla

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workbookDisplayName":  {             
            "type":"string",
            "defaultValue": "My Workbook",
            "metadata": {
                "description": "The friendly name for the workbook that is used in the Gallery or Saved List. Needs to be unique in the scope of the resource group and source" 
            }
        },
        "workbookType":  {             
            "type":"string",
            "defaultValue": "tsg",
            "metadata": {
                "description": "The gallery that the workbook will been shown under. Supported values include workbook, `tsg`, Azure Monitor, etc." 
            }
        },
        "workbookSourceId":  {             
            "type":"string",
            "defaultValue": "<insert-your-resource-id-here>",
            "metadata": {
                "description": "The id of resource instance to which the workbook will be associated" 
            }
        },
        "workbookId": {
            "type":"string",
            "defaultValue": "[newGuid()]",
            "metadata": {
                "description": "The unique guid for this workbook instance" 
            }
        }
    },    
    "resources": [
        {
            "name": "[parameters('workbookId')]",
            "type": "Microsoft.Insights/workbooks",
            "location": "[resourceGroup().location]",
            "kind": "shared",
            "apiVersion": "2018-06-17-preview",
            "dependsOn": [],
            "properties": {
                "displayName": "[parameters('workbookDisplayName')]",
                "serializedData": "{\"version\":\"Notebook/1.0\",\"items\":[{\"type\":1,\"content\":\"{\\\"json\\\":\\\"Hello World!\\\"}\",\"conditionalVisibility\":null}],\"isLocked\":false}",
                "version": "1.0",
                "sourceId": "[parameters('workbookSourceId')]",
                "category": "[parameters('workbookType')]"
            }
        }
    ],
    "outputs": {
        "workbookId": {
            "type": "string",
            "value": "[resourceId( 'Microsoft.Insights/workbooks', parameters('workbookId'))]"
        }
    }
}
```

### <a name="parameter-file"></a>Archivo de parámetros

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workbookDisplayName": {
            "value": "Sample Hello World workbook"
        },
      "workbookType": {
        "value": "workbook"
      },
      "workbookSourceId": {
        "value": "Azure Monitor"
      }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Otras plantillas de ejemplo para Azure Monitor](../resource-manager-samples.md).
* [Más información sobre los grupos de acciones](../visualize/workbooks-overview.md)