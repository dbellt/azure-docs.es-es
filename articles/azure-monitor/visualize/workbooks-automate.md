---
title: Libros de Azure Monitor y plantillas de Azure Resource Manager
description: Simplifique los informes complejos con libros de Azure Monitor parametrizados predefinidos y personalizados implementados a partir de plantillas de Azure Resource Manager
services: azure-monitor
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 04/30/2020
ms.openlocfilehash: 48766e51e3408e11c264b77c43b066f3fedc6c28
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110080513"
---
# <a name="programmatically-manage-workbooks"></a>Administración de libros mediante programación

Los propietarios de recursos tienen la opción de crear y administrar sus libros mediante programación a través de plantillas de Resource Manager.

Esto puede ser útil en escenarios como:
* Implementación de informes de análisis específicos de la organización o del dominio junto con las implementaciones de recursos. Por ejemplo, puede implementar libros de error y rendimiento específicos de la organización para las nuevas aplicaciones o máquinas virtuales.
* Implementación de informes estándar o paneles con los libros para los recursos existentes.

El libro se creará en el grupo o subgrupo de recursos deseado y con el contenido especificado en las plantillas de Resource Manager.

Hay dos tipos de recursos de libro que se pueden administrar mediante programación:
* [Plantillas de libro](#azure-resource-manager-template-for-deploying-a-workbook-template)
* [Instancias de libro](#azure-resource-manager-template-for-deploying-a-workbook-instance)

## <a name="azure-resource-manager-template-for-deploying-a-workbook-template"></a>Plantilla de Azure Resource Manager para implementar una plantilla de libro

1. Abra un libro que quiera implementar mediante programación.
2. Para cambiar el libro al modo de edición, haga clic en el elemento de la barra de herramientas _Editar_.
3. Abra el _Editor avanzado_ con el botón _</>_ de la barra de herramientas.
4. Asegúrese de que se encuentre en la pestaña _Plantilla de la galería_.

    ![Pestaña Plantilla de la galería](./media/workbooks-automate/gallery-template.png)
1. Copie el archivo JSON de la plantilla de la galería en el portapapeles.
2. A continuación, se muestra una plantilla de Azure Resource Manager de ejemplo que implementa una plantilla de libro en la galería de libros de Azure Monitor. Pegue el código JSON que copió en lugar de `<PASTE-COPIED-WORKBOOK_TEMPLATE_HERE>`. Se puede encontrar una plantilla de Azure Resource Manager de referencia que crea una plantilla de libro [aquí](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Documentation/ARM-template-for-creating-workbook-template).

    ```json
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "resourceName": {
                "type": "string",
                "defaultValue": "my-workbook-template",
                "metadata": {
                    "description": "The unique name for this workbook template instance"
                }
            }
        },
        "resources": [
            {
                "name": "[parameters('resourceName')]",
                "type": "microsoft.insights/workbooktemplates",
                "location": "[resourceGroup().location]",
                "apiVersion": "2019-10-17-preview",
                "dependsOn": [],
                "properties": {
                    "galleries": [
                        {
                            "name": "A Workbook Template",
                            "category": "Deployed Templates",
                            "order": 100,
                            "type": "workbook",
                            "resourceType": "Azure Monitor"
                        }
                    ],
                    "templateData": <PASTE-COPIED-WORKBOOK_TEMPLATE_HERE>
                }
            }
        ]
    }
    ```
1. En el objeto `galleries`, rellene las claves `name` y `category` con sus valores. Más información sobre los [parámetros](#parameters) en la sección siguiente.
2. Implemente esta plantilla de Azure Resource Manager mediante [Azure Portal](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template), la [interfaz de la línea de comandos](../../azure-resource-manager/templates/deploy-cli.md), [PowerShell](../../azure-resource-manager/templates/deploy-powershell.md), etc.
3. Abra Azure Portal y vaya a la galería de libros elegida en la plantilla de Azure Resource Manager. En la plantilla de ejemplo, vaya a la galería de libros de Azure Monitor:
    1. Abra Azure Portal y vaya a Azure Monitor.
    2. Abra `Workbooks` desde la tabla de contenido
    3. Busque la plantilla en la galería en la categoría `Deployed Templates` (será uno de los elementos púrpura).

### <a name="parameters"></a>Parámetros

|Parámetros                |Explicación                                                                                             |
|:-------------------------|:-------------------------------------------------------------------------------------------------------|
| `name`                   | Nombre del recurso de plantilla de libro en Azure Resource Manager.                                  |
|`type`                    | Siempre microsoft.insights/workbooktemplates                                                            |
| `location`               | Ubicación de Azure donde se creará el libro.                                               |
| `apiVersion`             | 2019-10-17 preview                                                                                     |
| `type`                   | Siempre microsoft.insights/workbooktemplates                                                            |
| `galleries`              | Conjunto de galerías en las que se va a mostrar esta plantilla de libro.                                                |
| `gallery.name`           | Nombre descriptivo de la plantilla de libro en la galería.                                             |
| `gallery.category`       | Grupo de la galería en el que se va a colocar la plantilla.                                                     |
| `gallery.order`          | Número que decide el orden en que se muestra la plantilla en una categoría de la galería. A menor orden mayor prioridad. |
| `gallery.resourceType`   | Tipo de recurso correspondiente a la galería. Normalmente, se trata de la cadena de tipo de recurso correspondiente al recurso (por ejemplo, microsoft.operationalinsights/workspaces). |
|`gallery.type`            | Conocido como tipo de libro, se trata de una clave única que diferencia la galería dentro de un tipo de recurso. Application Insights, por ejemplo, tiene los tipos `workbook` y `tsg` correspondientes a diferentes galerías de libros. |

### <a name="galleries"></a>Galerías

| Galería                                        | Tipo de recurso                                       | Tipo de libro |
|:-----------------------------------------------|:----------------------------------------------------|:--------------|
| Libros en Azure Monitor                     | `Azure Monitor`                                     | `workbook`    |
| VM Insights en Azure Monitor                   | `Azure Monitor`                                     | `vm-insights` |
| Libros en el área de trabajo de Log Analytics           | `microsoft.operationalinsights/workspaces`          | `workbook`    |
| Libros en Application Insights              | `microsoft.insights/components`                     | `workbook`    |
| Guías de solución de problemas de Application Insights | `microsoft.insights/components`                     | `tsg`         |
| Uso en Application Insights                  | `microsoft.insights/components`                     | `usage`       |
| Libros en el servicio Kubernetes                | `Microsoft.ContainerService/managedClusters`        | `workbook`    |
| Libros en grupos de recursos                   | `microsoft.resources/subscriptions/resourcegroups`  | `workbook`    |
| Libros en Azure Active Directory            | `microsoft.aadiam/tenant`                           | `workbook`    |
| VM Insights en máquinas virtuales                | `microsoft.compute/virtualmachines`                 | `insights`    |
| VM Insights en conjuntos de escalado de máquinas virtuales      | `microsoft.compute/virtualmachinescalesets`         | `insights`    |

## <a name="azure-resource-manager-template-for-deploying-a-workbook-instance"></a>Plantilla de Azure Resource Manager para implementar una instancia de libro

1. Abra un libro que desee implementar mediante programación.
2. Para cambiar el libro al modo de edición, haga clic en el elemento de la barra de herramientas _Editar_.
3. Abra el _Editor avanzado_ con el botón _</>_ de la barra de herramientas.
4. En el editor, cambie _Tipo de plantilla_ a _Plantilla de Resource Manager_.
5. La plantilla de Resource Manager para la creación se muestra en el editor. Copie el contenido y úselo tal cual o mézclelo con una plantilla más grande que también implemente el recurso de destino.

    ![Imagen en la que se muestra cómo obtener la plantilla de Resource Manager desde la interfaz de usuario del libro](./media/workbooks-automate/programmatic-template.png)

## <a name="sample-azure-resource-manager-template"></a>Plantilla de Azure Resource Manager de ejemplo
Esta plantilla muestra cómo implementar un libro sencillo que muestra un '¡Hola mundo!'
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

### <a name="template-parameters"></a>Parámetros de plantilla

| Parámetro | Explicación |
| :------------- |:-------------|
| `workbookDisplayName` | Nombre descriptivo para el libro que se usa en la galería o en la lista guardada. Debe ser único en el grupo de recursos y origen |
| `workbookType` | La galería en la que se mostrará el libro. Los valores admitidos son libro, `tsg`, Azure Monitor, etc. |
| `workbookSourceId` | Identificador de la instancia de recurso a la que se asociará el libro. El nuevo libro se mostrará relacionado con esta instancia de recurso, por ejemplo, en la tabla de contenido del recurso en _Libro_. Si desea que el libro aparezca en la galería de libros en Azure Monitor, use la cadena _Azure Monitor_ en lugar de un identificador de recurso. |
| `workbookId` | El GUID único para esta instancia de libro. Use _[newGuid()]_ para crear automáticamente un nuevo GUID. |
| `kind` | Se utiliza para especificar si el libro creado es compartido o privado. Use el valor _compartido_ para los libros compartidos y _usuario_ para los usuarios privados. |
| `location` | Ubicación de Azure donde se creará el libro. Use _[resourceGroup().location]_ para crearlo en la misma ubicación que el grupo de recursos |
| `serializedData` | Contiene el contenido o la carga que se va a usar en el libro. Use la plantilla de Resource Manager de la interfaz de usuario de los libros para obtener el valor |

### <a name="workbook-types"></a>Tipos de libros
Tipos de libros especifica el tipo de galería de libros en el que se mostrará la nueva instancia de libro. Las opciones incluyen:

| Tipo | Ubicación de la galería |
| :------------- |:-------------|
| `workbook` | El valor predeterminado que se usa en la mayoría de los informes, incluida la galería de libros de Application Insights, Azure Monitor, etc.  |
| `tsg` | La galería de guías de solución de problemas en Application Insights |
| `usage` | La galería _Más_ en _Uso_ en Application Insights |

### <a name="working-with-json-formatted-workbook-data-in-the-serializeddata-template-parameter"></a>Trabajo con datos de libro con formato JSON en el parámetro de plantilla serializedData

Al exportar una plantilla de Azure Resource Manager para un libro de Azure, a menudo hay vínculos de recursos fijos insertados dentro del parámetro de plantilla `serializedData` exportado. Estos incluyen valores potencialmente confidenciales, como el identificador de suscripción y el nombre del grupo de recursos, y otros tipos de identificadores de recursos.

En el ejemplo siguiente se muestra la personalización de una plantilla de Azure Resource Manager de libro exportada, sin recurrir a la manipulación de cadenas. El patrón que se muestra en este ejemplo está pensado para trabajar con los datos sin modificar tal como se exportan desde Azure Portal. También es un procedimiento recomendado para enmascarar los valores confidenciales insertados al administrar libros mediante programación; por ello, el identificador de suscripción y el grupo de recursos se han enmascarado aquí. No se realizaron otras modificaciones en el valor `serializedData` entrante sin procesar.

```json
{
  "contentVersion": "1.0.0.0",
  "parameters": {
    "workbookDisplayName": {
      "type": "string"
    },
    "workbookSourceId": {
      "type": "string",
      "defaultValue": "[resourceGroup().id]"
    },
    "workbookId": {
      "type": "string",
      "defaultValue": "[newGuid()]"
    }
  },
  "variables": {
    // serializedData from original exported Azure Resource Manager template
    "serializedData": "{\"version\":\"Notebook/1.0\",\"items\":[{\"type\":1,\"content\":{\"json\":\"Replace with Title\"},\"name\":\"text - 0\"},{\"type\":3,\"content\":{\"version\":\"KqlItem/1.0\",\"query\":\"{\\\"version\\\":\\\"ARMEndpoint/1.0\\\",\\\"data\\\":null,\\\"headers\\\":[],\\\"method\\\":\\\"GET\\\",\\\"path\\\":\\\"/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups\\\",\\\"urlParams\\\":[{\\\"key\\\":\\\"api-version\\\",\\\"value\\\":\\\"2019-06-01\\\"}],\\\"batchDisabled\\\":false,\\\"transformers\\\":[{\\\"type\\\":\\\"jsonpath\\\",\\\"settings\\\":{\\\"tablePath\\\":\\\"$..*\\\",\\\"columns\\\":[]}}]}\",\"size\":0,\"queryType\":12,\"visualization\":\"map\",\"tileSettings\":{\"showBorder\":false},\"graphSettings\":{\"type\":0},\"mapSettings\":{\"locInfo\":\"AzureLoc\",\"locInfoColumn\":\"location\",\"sizeSettings\":\"location\",\"sizeAggregation\":\"Count\",\"opacity\":0.5,\"legendAggregation\":\"Count\",\"itemColorSettings\":null}},\"name\":\"query - 1\"}],\"isLocked\":false,\"fallbackResourceIds\":[\"/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/XXXXXXX\"]}",

    // parse the original into a JSON object, so that it can be manipulated
    "parsedData": "[json(variables('serializedData'))]",

    // create new JSON objects that represent only the items/properties to be modified
    "updatedTitle": {
      "content":{
        "json": "[concat('Resource Group Regions in subscription \"', subscription().displayName, '\"')]"
      }
    },
    "updatedMap": {
      "content": {
        "path": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups')]"
      }
    },

    // the union function applies the updates to the original data
    "updatedItems": [
      "[union(variables('parsedData')['items'][0], variables('updatedTitle'))]",
      "[union(variables('parsedData')['items'][1], variables('updatedMap'))]"
    ],

    // copy to a new workbook object, with the updated items
    "updatedWorkbookData": {
      "version": "[variables('parsedData')['version']]",
      "items": "[variables('updatedItems')]",
      "isLocked": "[variables('parsedData')['isLocked']]",
      "fallbackResourceIds": ["[parameters('workbookSourceId')]"]
    },

    // convert back to an encoded string
    "reserializedData": "[string(variables('updatedWorkbookData'))]"
  },
  "resources": [
    {
      "name": "[parameters('workbookId')]",
      "type": "microsoft.insights/workbooks",
      "location": "[resourceGroup().location]",
      "apiVersion": "2018-06-17-preview",
      "dependsOn": [],
      "kind": "shared",
      "properties": {
        "displayName": "[parameters('workbookDisplayName')]",
        "serializedData": "[variables('reserializedData')]",
        "version": "1.0",
        "sourceId": "[parameters('workbookSourceId')]",
        "category": "workbook"
      }
    }
  ],
  "outputs": {
    "workbookId": {
      "type": "string",
      "value": "[resourceId( 'microsoft.insights/workbooks', parameters('workbookId'))]"
    }
  },
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#"
}
```

En este ejemplo, los pasos siguientes facilitaron la personalización de una plantilla de Azure Resource Manager exportada:
1. Exporte el libro como plantilla de Azure Resource Manager como se explica en la sección anterior.
2. En la sección `variables` de la plantilla:
    1. Analice el valor `serializedData` en una variable de objeto JSON, que crea una estructura JSON que incluye una matriz de elementos que representan el contenido del libro.
    2. Cree nuevos objetos JSON que representen solo los elementos o propiedades que se modificarán.
    3. Proyecte un nuevo conjunto de elementos de contenido JSON (`updatedItems`), mediante la función `union()` para aplicar las modificaciones a los elementos JSON originales.
    4. Cree un nuevo objeto de libro, `updatedWorkbookData`, que contenga `updatedItems` y los datos `version`/`isLocked` de los datos analizados originales, así como un conjunto corregido de `fallbackResourceIds`.
    5. Vuelva a serializar el nuevo contenido JSON en una nueva variable de cadena, `reserializedData`.
3. Use la nueva variable `reserializedData` en lugar de la propiedad `serializedData` original.
4. Implemente el nuevo recurso de libro con la plantilla de Azure Resource Manager actualizada.

### <a name="limitations"></a>Limitaciones
Por motivos técnicos, este mecanismo no se puede usar para crear instancias de libro en la galería de _Libros_ de Application Insights. Estamos trabajando actualmente para poder resolver este incidente. Mientras tanto, se recomienda usar la galería de la guía de solución de problemas (workbookType: `tsg`) para implementar libros relacionados de Application Insights.

## <a name="next-steps"></a>Pasos siguientes

Explore cómo se usan los libros para potenciar la nueva [experiencia de Conclusiones de Storage](../insights/storage-insights-overview.md).
