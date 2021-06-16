---
title: Actividad Until en Azure Data Factory
description: La actividad Until ejecuta un conjunto de actividades en un bucle hasta que la condición asociada a la actividad la evalúa como "true" o se agota el tiempo de espera.
author: chez-charlie
ms.author: chez
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b69910daab3172a6c7ba2cd134cb9ed41ebaaa79
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110675011"
---
# <a name="until-activity-in-azure-data-factory"></a>Actividad Until en Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

La actividad Until proporciona la misma funcionalidad que una estructura de bucle Do-Until en lenguajes de programación. Ejecuta un conjunto de actividades en un bucle hasta que la condición asociada a la actividad la evalúa como "true". Puede especificar un valor de tiempo de espera para la actividad Until en Data Factory. 

## <a name="syntax"></a>Sintaxis

```json
{
    "type": "Until",
    "typeProperties": {
        "expression":  {
            "value":  "<expression that evaluates to true or false>", 
            "type": "Expression"
        },
        "timeout": "<time out for the loop. for example: 00:01:00 (1 minute)>",
        "activities": [
            {
                "<Activity 1 definition>"
            },
            {
                "<Activity 2 definition>"
            },
            {
                "<Activity N definition>"
            }
        ]
    },
    "name": "MyUntilActivity"
}

```

## <a name="type-properties"></a>Propiedades de tipo

Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | --------
name | Nombre de la actividad `Until`. | String | Sí
type | Debe establecerse en **Until**. | String | Sí
expresión | Expresión que debe evaluarse como "true" o "false" | Expresión.  | Sí
timeout | El tiempo de espera del bucle Do-Until se agota transcurrido el tiempo especificado aquí. | String. `d.hh:mm:ss` (o) `hh:mm:ss`. El valor predeterminado es de 7 días. El valor máximo es 90 días. | No
Actividades | Conjunto de actividades que se ejecutan hasta que la expresión se evalúa como `true`. | Matriz de actividades. |  Sí

## <a name="example-1"></a>Ejemplo 1

> [!NOTE]
> En esta sección se proporcionan definiciones JSON y comandos de PowerShell de ejemplo para ejecutarlos en la canalización. Para ver una guía con instrucciones paso a paso para crear una canalización de Data Factory con definiciones de JSON y Azure PowerShell, consulte [Tutorial: Creación de una factoría de datos y una canalización con PowerShell](quickstart-create-data-factory-powershell.md).

### <a name="pipeline-with-until-activity"></a>Canalización con actividad Until
En este ejemplo, la canalización tiene dos actividades: **Until** y **Wait**. La actividad Wait espera durante el período de tiempo especificado antes de ejecutar la actividad Web en el bucle. Para obtener más información sobre las expresiones y funciones de Data Factory, consulte [Expresiones y funciones de Azure Data Factory](control-flow-expression-language-functions.md). 

```json
{
    "name": "DoUntilPipeline",
    "properties": {
        "activities": [
            {
                "type": "Until",
                "typeProperties": {
                    "expression": {
                        "value": "@equals('Failed', coalesce(body('MyUnauthenticatedActivity')?.status, actions('MyUnauthenticatedActivity')?.status, 'null'))",
                        "type": "Expression"
                    },
                    "timeout": "00:00:01",
                    "activities": [
                        {
                            "name": "MyUnauthenticatedActivity",
                            "type": "WebActivity",
                            "typeProperties": {
                                "method": "get",
                                "url": "https://www.fake.com/",
                                "headers": {
                                    "Content-Type": "application/json"
                                }
                            },
                            "dependsOn": [
                                {
                                    "activity": "MyWaitActivity",
                                    "dependencyConditions": [ "Succeeded" ]
                                }
                            ]
                        },
                        {
                            "type": "Wait",
                            "typeProperties": {
                                "waitTimeInSeconds": 1
                            },
                            "name": "MyWaitActivity"
                        }
                    ]
                },
                "name": "MyUntilActivity"
            }
        ]
    }
}

```

## <a name="example-2"></a>Ejemplo 2 
La canalización de esta muestra copia los datos de una carpeta de entrada a una carpeta de salida en un bucle. El bucle finaliza cuando el valor del parámetro repeat está establecido en "false" o se agota el tiempo de espera transcurrido un minuto.   

### <a name="pipeline-with-until-activity-adfv2quickstartpipelinejson"></a>Canalización con actividad Until (Adfv2QuickStartPipeline.json)

```json
{
    "name": "Adfv2QuickStartPipeline",
    "properties": {
        "activities": [
            {
                "type": "Until",
                "typeProperties": {
                    "expression":  {
                        "value":  "@equals('false', pipeline().parameters.repeat)", 
                        "type": "Expression"
                    },
                    "timeout": "00:01:00",
                    "activities": [
                        {
                            "name": "CopyFromBlobToBlob",
                            "type": "Copy",
                            "inputs": [
                                {
                                    "referenceName": "BlobDataset",
                                    "parameters": {
                                        "path": "@pipeline().parameters.inputPath"
                                    },
                                    "type": "DatasetReference"
                                }
                            ],
                            "outputs": [
                                {
                                    "referenceName": "BlobDataset",
                                    "parameters": {
                                        "path": "@pipeline().parameters.outputPath"
                                    },
                                    "type": "DatasetReference"
                                }
                            ],
                            "typeProperties": {
                                "source": {
                                    "type": "BlobSource"
                                },
                                "sink": {
                                    "type": "BlobSink"
                                }
                            },
                            "policy": {
                                "retry": 1,
                                "timeout": "00:10:00",
                                "retryIntervalInSeconds": 60
                            }
                        }
                    ]
                },
                "name": "MyUntilActivity"
            }
        ],
        "parameters": {
            "inputPath": {
                "type": "String"
            },
            "outputPath": {
                "type": "String"
            },
            "repeat": {
                "type": "String"
            }                        
        }        
    }
}

```


### <a name="azure-storage-linked-service-azurestoragelinkedservicejson"></a>Servicio vinculado de Azure Storage (AzureStorageLinkedService.json)

```json
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<Azure Storage account name>;AccountKey=<Azure Storage account key>"
        }
    }
}
```

### <a name="parameterized-azure-blob-dataset-blobdatasetjson"></a>Conjunto de datos de blob de Azure con parámetros (BlobDataset.json)
La canalización establece **folderPath** en el valor de los parámetros **outputPath1** o **outputPath2** de la canalización. 

```json
{
    "name": "BlobDataset",
    "properties": {
        "type": "AzureBlob",
        "typeProperties": {
            "folderPath": {
                "value": "@{dataset().path}",
                "type": "Expression"
            }
        },
        "linkedServiceName": {
            "referenceName": "AzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "path": {
                "type": "String"
            }
        }
    }
}
```

### <a name="pipeline-parameter-json-pipelineparametersjson"></a>Parámetro de canalización JSON (PipelineParameters.json)

```json
{
    "inputPath": "adftutorial/input",
    "outputPath": "adftutorial/outputUntil",
    "repeat": "true"
}
```

### <a name="powershell-commands"></a>Comandos de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Estos comandos presuponen que ha guardado los archivos JSON en la siguiente carpeta: C:\ADF. 

```powershell
Connect-AzAccount
Select-AzSubscription "<Your subscription name>"

$resourceGroupName = "<Resource Group Name>"
$dataFactoryName = "<Data Factory Name. Must be globally unique>";
Remove-AzDataFactoryV2 $dataFactoryName -ResourceGroupName $resourceGroupName -force


Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location "East US" -Name $dataFactoryName
Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureStorageLinkedService" -DefinitionFile "C:\ADF\AzureStorageLinkedService.json"
Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "BlobDataset" -DefinitionFile "C:\ADF\BlobDataset.json"
Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "Adfv2QuickStartPipeline" -DefinitionFile "C:\ADF\Adfv2QuickStartPipeline.json"
$runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName "Adfv2QuickStartPipeline" -ParameterFile C:\ADF\PipelineParameters.json

while ($True) {
    $run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $resourceGroupName -DataFactoryName $DataFactoryName -PipelineRunId $runId

    if ($run) {
        if ($run.Status -ne 'InProgress') {
            Write-Host "Pipeline run finished. The status is: " $run.Status -foregroundcolor "Yellow"
            $run
            break
        }
        Write-Host  "Pipeline is running...status: InProgress" -foregroundcolor "Yellow"
        Write-Host "Activity run details:" -foregroundcolor "Yellow"
        $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
        $result

        Write-Host "Activity 'Output' section:" -foregroundcolor "Yellow"
        $result.Output -join "`r`n"
    }

    Start-Sleep -Seconds 15
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
