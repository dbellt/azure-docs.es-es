---
title: Actividad de establecimiento de variables en Azure Data Factory
description: Aprenda cómo usar la actividad de establecimiento de variables para establecer el valor de una variable existente definida en una canalización de Data Factory
ms.service: data-factory
ms.topic: conceptual
ms.date: 04/07/2020
author: chez-charlie
ms.author: chez
ms.reviewer: jburchel
ms.openlocfilehash: 5110160234131bd105ac860efe818506db48f6b0
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107903927"
---
# <a name="set-variable-activity-in-azure-data-factory"></a>Actividad de establecimiento de variables en Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Utilice la actividad de establecimiento de variables para establecer el valor de una variable existente de tipo cadena, booleano o matriz definida en una canalización de Data Factory.

## <a name="type-properties"></a>Propiedades de tipo

Propiedad | Descripción | Obligatorio
-------- | ----------- | --------
name | Nombre de la actividad en la canalización | sí
description | Texto que describe para qué se usa la actividad | no
type | Debe establecerse en **SetVariable** | sí
value | Literal de cadena o valor de objeto de expresión al que se asigna la variable | sí
variableName | Nombre de la variable que esta actividad establece | sí

## <a name="incrementing-a-variable"></a>Incremento de una variable

Un escenario común relacionado con variables en Azure Data Factory es el uso de una variable como iterador dentro de una actividad until o foreach. En una actividad de establecimiento de variables no se puede hacer referencia a la variable que se establece en el campo `value`. Para solucionar esta limitación, establezca una variable temporal y, a continuación, cree una segunda actividad de establecimiento de variables. La segunda actividad de establecimiento de variables establece el valor del iterador en la variable temporal. 

A continuación se muestra un ejemplo de este patrón:

![Incremento de variable](media/control-flow-set-variable-activity/increment-variable.png "Incrementar variable")

``` json
{
    "name": "pipeline3",
    "properties": {
        "activities": [
            {
                "name": "Set I",
                "type": "SetVariable",
                "dependsOn": [
                    {
                        "activity": "Increment J",
                        "dependencyConditions": [
                            "Succeeded"
                        ]
                    }
                ],
                "userProperties": [],
                "typeProperties": {
                    "variableName": "i",
                    "value": {
                        "value": "@variables('j')",
                        "type": "Expression"
                    }
                }
            },
            {
                "name": "Increment J",
                "type": "SetVariable",
                "dependsOn": [],
                "userProperties": [],
                "typeProperties": {
                    "variableName": "j",
                    "value": {
                        "value": "@string(add(int(variables('i')), 1))",
                        "type": "Expression"
                    }
                }
            }
        ],
        "variables": {
            "i": {
                "type": "String",
                "defaultValue": "0"
            },
            "j": {
                "type": "String",
                "defaultValue": "0"
            }
        },
        "annotations": []
    }
}
```


## <a name="next-steps"></a>Pasos siguientes
Obtenga información acerca de una actividad de flujo de control relacionada compatible con Data Factory: 

- [Actividad de anexión de variables](control-flow-append-variable-activity.md)
