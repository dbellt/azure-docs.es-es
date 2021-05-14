---
title: Actividad ForEach en Azure Data Factory
description: La actividad ForEach define un flujo de control repetido en la canalización. Se usa para iterar una colección y ejecuta actividades especificadas.
author: chez-charlie
ms.author: chez
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/23/2019
ms.openlocfilehash: 28c67640a65e44fb9c6d6791229796c614993fa1
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107906303"
---
# <a name="foreach-activity-in-azure-data-factory"></a>Actividad ForEach en Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

La actividad ForEach define un flujo de control repetido en la canalización. Esta actividad se usa para iterar una colección y ejecuta las actividades especificadas en un bucle. La implementación del bucle de esta actividad es similar a la estructura de bucle ForEach de los lenguajes de programación.

## <a name="syntax"></a>Sintaxis
Las propiedades se describen más adelante en este artículo. La propiedad items es la colección y cada elemento de la colección se reconoce por usar `@item()`, como se muestra en la sintaxis siguiente:  

```json
{  
   "name":"MyForEachActivityName",
   "type":"ForEach",
   "typeProperties":{  
      "isSequential":"true",
        "items": {
            "value": "@pipeline().parameters.mySinkDatasetFolderPathCollection",
            "type": "Expression"
        },
      "activities":[  
         {  
            "name":"MyCopyActivity",
            "type":"Copy",
            "typeProperties":{  
               ...
            },
            "inputs":[  
               {  
                  "referenceName":"MyDataset",
                  "type":"DatasetReference",
                  "parameters":{  
                     "MyFolderPath":"@pipeline().parameters.mySourceDatasetFolderPath"
                  }
               }
            ],
            "outputs":[  
               {  
                  "referenceName":"MyDataset",
                  "type":"DatasetReference",
                  "parameters":{  
                     "MyFolderPath":"@item()"
                  }
               }
            ]
         }
      ]
   }
}

```

## <a name="type-properties"></a>Propiedades de tipo

Propiedad | Descripción | Valores permitidos | Obligatorio
-------- | ----------- | -------------- | --------
name | Nombre de la actividad for-each. | String | Sí
type | Se debe establecer en **ForEach** | String | Sí
isSequential | Especifica si el bucle se debe ejecutar en secuencia o en paralelo.  Se puede ejecutar un máximo de 20 iteraciones de bucle a la vez en paralelo. Por ejemplo, si tiene una actividad ForEach que itera una actividad de copia con 10 conjuntos de datos de origen y receptor distintos con **isSequential** establecido en False, todas las copias se ejecutan a la vez. El valor predeterminado es False. <br/><br/> Si "isSequential" está establecido en False, asegúrese de que haya una configuración correcta para ejecutar varios archivos ejecutables. De lo contrario, esta propiedad se debe usar con precaución para no incurrir en conflictos de escritura. Para más información, consulte la sección [Ejecución en paralelo](#parallel-execution). | Boolean | No. El valor predeterminado es False.
batchCount | Número de lotes que se usará para controlar el número de la ejecución en paralelo (cuando isSequential está establecido en false). Este es el límite de simultaneidad superior, pero la actividad for-each no siempre se ejecutará en este número. | Entero (50 como máximo) | No. El valor predeterminado es 20.
Elementos | Una expresión que devuelve una matriz JSON que se iterará. | Expresión (que devuelve una matriz JSON) | Sí
Actividades | Las actividades que se ejecutarán. | Lista de actividades | Sí

## <a name="parallel-execution"></a>Ejecución en paralelo
Si **isSequential** está establecido en false, la actividad itera en paralelo con un máximo de 20 iteraciones simultáneas. Esta configuración se debe usar con precaución. Si las iteraciones simultáneas escriben en la misma carpeta pero en distintos archivos, este enfoque es correcto. Si las iteraciones simultáneas escriben al mismo tiempo exactamente en el mismo archivo, es más probable que este enfoque provoque un error. 

## <a name="iteration-expression-language"></a>Lenguaje de expresión de iteración
En la actividad ForEach, proporcione una matriz que se iterará para la propiedad **items**. Use `@item()` para iterar una sola enumeración en la actividad ForEach. Por ejemplo, si la propiedad **items** es una matriz: [1, 2, 3], `@item()` devuelve 1 en la primera iteración, 2 en la segunda y 3 en la tercera. También puede usar `@range(0,10)` como expresión para iterar diez veces comenzando con 0 y terminando con 9.

## <a name="iterating-over-a-single-activity"></a>Iteración de una sola actividad
**Escenario:** Copia del mismo archivo de origen en un blob de Azure a varios archivos de destino en un blob de Azure.

### <a name="pipeline-definition"></a>Definición de la canalización

```json
{
    "name": "<MyForEachPipeline>",
    "properties": {
        "activities": [
            {
                "name": "<MyForEachActivity>",
                "type": "ForEach",
                "typeProperties": {
                    "isSequential": "true",
                    "items": {
                        "value": "@pipeline().parameters.mySinkDatasetFolderPath",
                        "type": "Expression"
                    },
                    "activities": [
                        {
                            "name": "MyCopyActivity",
                            "type": "Copy",
                            "typeProperties": {
                                "source": {
                                    "type": "BlobSource",
                                    "recursive": "false"
                                },
                                "sink": {
                                    "type": "BlobSink",
                                    "copyBehavior": "PreserveHierarchy"
                                }
                            },
                            "inputs": [
                                {
                                    "referenceName": "<MyDataset>",
                                    "type": "DatasetReference",
                                    "parameters": {
                                        "MyFolderPath": "@pipeline().parameters.mySourceDatasetFolderPath"
                                    }
                                }
                            ],
                            "outputs": [
                                {
                                    "referenceName": "MyDataset",
                                    "type": "DatasetReference",
                                    "parameters": {
                                        "MyFolderPath": "@item()"
                                    }
                                }
                            ]
                        }
                    ]
                }
            }
        ],
        "parameters": {
            "mySourceDatasetFolderPath": {
                "type": "String"
            },
            "mySinkDatasetFolderPath": {
                "type": "String"
            }
        }
    }
}

```

### <a name="blob-dataset-definition"></a>Definición del conjunto de datos de blob

```json
{  
   "name":"<MyDataset>",
   "properties":{  
      "type":"AzureBlob",
      "typeProperties":{  
         "folderPath":{  
            "value":"@dataset().MyFolderPath",
            "type":"Expression"
         }
      },
      "linkedServiceName":{  
         "referenceName":"StorageLinkedService",
         "type":"LinkedServiceReference"
      },
      "parameters":{  
         "MyFolderPath":{  
            "type":"String"
         }
      }
   }
}

```

### <a name="run-parameter-values"></a>Valores de parámetro de ejecución

```json
{
    "mySourceDatasetFolderPath": "input/",
    "mySinkDatasetFolderPath": [ "outputs/file1", "outputs/file2" ]
}

```

## <a name="iterate-over-multiple-activities"></a>Iteración de varias actividades
Es posible iterar varias actividades (por ejemplo: actividades de copia y web) en una actividad ForEach. En este escenario, se recomienda abstraer varias actividades en una canalización independiente. Luego puede usar la [actividad ExecutePipeline](control-flow-execute-pipeline-activity.md) en la canalización con la actividad ForEach para invocar la canalización independiente con varias actividades. 


### <a name="syntax"></a>Sintaxis

```json
{
  "name": "masterPipeline",
  "properties": {
    "activities": [
      {
        "type": "ForEach",
        "name": "<MyForEachMultipleActivities>"
        "typeProperties": {
          "isSequential": true,
          "items": {
            ...
          },
          "activities": [
            {
              "type": "ExecutePipeline",
              "name": "<MyInnerPipeline>"
              "typeProperties": {
                "pipeline": {
                  "referenceName": "<copyHttpPipeline>",
                  "type": "PipelineReference"
                },
                "parameters": {
                  ...
                },
                "waitOnCompletion": true
              }
            }
          ]
        }
      }
    ],
    "parameters": {
      ...
    }
  }
}

```

### <a name="example"></a>Ejemplo
**Escenario:** Iteración de una canalización interna dentro de una actividad ForEach con la actividad de ejecución de canalización. La canalización interna copia con definiciones de esquema parametrizadas.

#### <a name="master-pipeline-definition"></a>Definición de la canalización principal

```json
{
  "name": "masterPipeline",
  "properties": {
    "activities": [
      {
        "type": "ForEach",
        "name": "MyForEachActivity",
        "typeProperties": {
          "isSequential": true,
          "items": {
            "value": "@pipeline().parameters.inputtables",
            "type": "Expression"
          },
          "activities": [
            {
              "type": "ExecutePipeline",
              "typeProperties": {
                "pipeline": {
                  "referenceName": "InnerCopyPipeline",
                  "type": "PipelineReference"
                },
                "parameters": {
                  "sourceTableName": {
                    "value": "@item().SourceTable",
                    "type": "Expression"
                  },
                  "sourceTableStructure": {
                    "value": "@item().SourceTableStructure",
                    "type": "Expression"
                  },
                  "sinkTableName": {
                    "value": "@item().DestTable",
                    "type": "Expression"
                  },
                  "sinkTableStructure": {
                    "value": "@item().DestTableStructure",
                    "type": "Expression"
                  }
                },
                "waitOnCompletion": true
              },
              "name": "ExecuteCopyPipeline"
            }
          ]
        }
      }
    ],
    "parameters": {
      "inputtables": {
        "type": "Array"
      }
    }
  }
}

```

#### <a name="inner-pipeline-definition"></a>Definición de la canalización interna

```json
{
  "name": "InnerCopyPipeline",
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "SqlSource",
            }
          },
          "sink": {
            "type": "SqlSink"
          }
        },
        "name": "CopyActivity",
        "inputs": [
          {
            "referenceName": "sqlSourceDataset",
            "parameters": {
              "SqlTableName": {
                "value": "@pipeline().parameters.sourceTableName",
                "type": "Expression"
              },
              "SqlTableStructure": {
                "value": "@pipeline().parameters.sourceTableStructure",
                "type": "Expression"
              }
            },
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "sqlSinkDataset",
            "parameters": {
              "SqlTableName": {
                "value": "@pipeline().parameters.sinkTableName",
                "type": "Expression"
              },
              "SqlTableStructure": {
                "value": "@pipeline().parameters.sinkTableStructure",
                "type": "Expression"
              }
            },
            "type": "DatasetReference"
          }
        ]
      }
    ],
    "parameters": {
      "sourceTableName": {
        "type": "String"
      },
      "sourceTableStructure": {
        "type": "String"
      },
      "sinkTableName": {
        "type": "String"
      },
      "sinkTableStructure": {
        "type": "String"
      }
    }
  }
}

```

#### <a name="source-dataset-definition"></a>Definición del conjunto de datos de origen

```json
{
  "name": "sqlSourceDataset",
  "properties": {
    "type": "SqlServerTable",
    "typeProperties": {
      "tableName": {
        "value": "@dataset().SqlTableName",
        "type": "Expression"
      }
    },
    "structure": {
      "value": "@dataset().SqlTableStructure",
      "type": "Expression"
    },
    "linkedServiceName": {
      "referenceName": "sqlserverLS",
      "type": "LinkedServiceReference"
    },
    "parameters": {
      "SqlTableName": {
        "type": "String"
      },
      "SqlTableStructure": {
        "type": "String"
      }
    }
  }
}

```

#### <a name="sink-dataset-definition"></a>Definición del conjunto de datos de receptor

```json
{
  "name": "sqlSinkDataSet",
  "properties": {
    "type": "AzureSqlTable",
    "typeProperties": {
      "tableName": {
        "value": "@dataset().SqlTableName",
        "type": "Expression"
      }
    },
    "structure": {
      "value": "@dataset().SqlTableStructure",
      "type": "Expression"
    },
    "linkedServiceName": {
      "referenceName": "azureSqlLS",
      "type": "LinkedServiceReference"
    },
    "parameters": {
      "SqlTableName": {
        "type": "String"
      },
      "SqlTableStructure": {
        "type": "String"
      }
    }
  }
}

```

#### <a name="master-pipeline-parameters"></a>Parámetros de la canalización principal
```json
{
    "inputtables": [
        {
            "SourceTable": "department",
            "SourceTableStructure": [
              {
                "name": "departmentid",
                "type": "int"
              },
              {
                "name": "departmentname",
                "type": "string"
              }
            ],
            "DestTable": "department2",
            "DestTableStructure": [
              {
                "name": "departmentid",
                "type": "int"
              },
              {
                "name": "departmentname",
                "type": "string"
              }
            ]
        }
    ]
    
}
```

## <a name="aggregating-outputs"></a>Agregación de salidas

Para agregar salidas de la actividad __foreach__, utilice _Variables_ y la actividad _Append Variable_.

Primero, declare una _variable_`array` en la canalización. A continuación, invoque la actividad _Append Variable_ dentro de cada bucle __foreach__. Posteriormente, puede recuperar la agregación de la matriz.

## <a name="limitations-and-workarounds"></a>Limitaciones y soluciones alternativas

A continuación se indican algunas de las limitaciones de la actividad ForEach y las soluciones alternativas sugeridas.

| Limitación | Solución alternativa |
|---|---|
| No se puede anidar un bucle ForEach dentro de otro bucle ForEach (o un bucle Until). | Diseñar una canalización de dos niveles, donde la canalización externa con el bucle ForEach exterior recorre en iteración una canalización interna con el bucle anidado. |
| La actividad ForEach tiene un valor máximo de `batchCount` de 50 para procesamiento en paralelo y un máximo de 100 000 elementos. | Diseñar una canalización de dos niveles, donde la canalización externa con la actividad ForEach recorre en iteración una canalización interna. |
| SetVariable no se puede utilizar dentro de una actividad ForEach que se ejecuta en paralelo, ya que las variables son globales para toda la canalización, no se limitan a una instrucción ForEach o a cualquier otra actividad. | Considere la posibilidad de usar una instrucción ForEach secuencial o ejecutar la canalización dentro de ForEach (variable/parámetro administrados en la canalización secundaria).|
| | |

## <a name="next-steps"></a>Pasos siguientes
Consulte otras actividades de flujo de control compatibles con Data Factory: 

- [Actividad de ejecución de canalización](control-flow-execute-pipeline-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
- [Actividad Lookup](control-flow-lookup-activity.md)
- [Actividad web](control-flow-web-activity.md)
