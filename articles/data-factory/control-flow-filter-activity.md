---
title: Actividad de filtro en Azure Data Factory
description: La actividad de filtro filtra las entradas.
author: chez-charlie
ms.author: chez
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/04/2018
ms.openlocfilehash: 6a488fff5e5c5f75697e787cdcede4de080a04ba
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107906281"
---
# <a name="filter-activity-in-azure-data-factory"></a>Actividad de filtro en Azure Data Factory
Puede usar una actividad de filtro en una canalización para aplicar una expresión de filtro a una matriz de entrada. 
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

## <a name="syntax"></a>Sintaxis

```json
{
    "name": "MyFilterActivity",
    "type": "filter",
    "typeProperties": {
        "condition": "<condition>",
        "items": "<input array>"
    }
}
```

## <a name="type-properties"></a>Propiedades de tipo

Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | --------
name | Nombre de la actividad `Filter`. | String | Sí
type | Se debe establecer en **filter**. | String | Sí
condición | Condición que se usará para filtrar la entrada. | Expression | Yes
items | Matriz de entrada en la que se debe aplicar el filtro. | Expression | Sí

## <a name="example"></a>Ejemplo

En este ejemplo, la canalización tiene dos actividades: **Filtro** y **Para cada uno**. La actividad de filtro está configurada para filtrar la matriz de entrada correspondiente a los elementos con un valor mayor que 3. La actividad "Para cada uno" recorre en iteración los valores filtrados y establece la variable **test** en el valor actual.

```json
{
    "name": "PipelineName",
    "properties": {
        "activities": [{
                "name": "MyFilterActivity",
                "type": "filter",
                "typeProperties": {
                    "condition": "@greater(item(),3)",
                    "items": "@pipeline().parameters.inputs"
                }
            },
            {
            "name": "MyForEach",
            "type": "ForEach",
            "dependsOn": [
                {
                    "activity": "MyFilterActivity",
                    "dependencyConditions": [
                        "Succeeded"
                    ]
                }
            ],
            "userProperties": [],
            "typeProperties": {
                "items": {
                    "value": "@activity('MyFilterActivity').output.value",
                    "type": "Expression"
                },
                "isSequential": "false",
                "batchCount": 1,
                "activities": [
                    {
                        "name": "Set Variable1",
                        "type": "SetVariable",
                        "dependsOn": [],
                        "userProperties": [],
                        "typeProperties": {
                            "variableName": "test",
                            "value": {
                                "value": "@string(item())",
                                "type": "Expression"
                            }
                        }
                    }
                ]
            }
        }],
        "parameters": {
            "inputs": {
                "type": "Array",
                "defaultValue": [1, 2, 3, 4, 5, 6]
            }
        },
        "variables": {
            "test": {
                "type": "String"
            }
        },
        "annotations": []
    }
}
```

## <a name="next-steps"></a>Pasos siguientes
Consulte otras actividades de flujo de control compatibles con Data Factory: 

- [Actividad If Condition](control-flow-if-condition-activity.md)
- [Actividad de ejecución de canalización](control-flow-execute-pipeline-activity.md)
- [Para cada actividad](control-flow-for-each-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
- [Actividad Lookup](control-flow-lookup-activity.md)
- [Actividad web](control-flow-web-activity.md)
- [Actividad Until](control-flow-until-activity.md)
