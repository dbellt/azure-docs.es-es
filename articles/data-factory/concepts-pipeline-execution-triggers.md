---
title: Ejecución y desencadenadores de canalización en Azure Data Factory
description: En este artículo se indica cómo ejecutar una canalización de Azure Data Factory a petición o mediante la creación de un desencadenador.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 07/05/2018
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e0696d648fa176314c4ed73eb68f4edbd73c32a2
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110675147"
---
# <a name="pipeline-execution-and-triggers-in-azure-data-factory"></a>Ejecución y desencadenadores de canalización en Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-scheduling-and-execution.md)
> * [Versión actual](concepts-pipeline-execution-triggers.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Una _ejecución de canalización_ en Azure Data Factory define una instancia de dicha ejecución. Por ejemplo, supongamos que tiene una canalización que se ejecuta las 8:00 a. m., 9:00 a. m. y 10:00 a. m. En este caso, hay tres ejecuciones independientes de la canalización o ejecuciones de canalización. Cada canalización ejecutar tiene un identificador de canalización único. El identificador de ejecución es un GUID que define de forma exclusiva una ejecución de canalización concreta.

Normalmente las instancias de ejecuciones de canalización se crean al pasar argumentos a parámetros que se definen en las canalizaciones. Puede ejecutar una canalización manualmente o mediante un _desencadenador_. En este artículo se proporcionan detalles sobre ambas maneras de ejecutar una canalización.

## <a name="manual-execution-on-demand"></a>Ejecución manual (a petición)

La ejecución manual de una canalización también se conoce como ejecución _a petición_.

Por ejemplo, supongamos que tiene una canalización básica denominada **copyPipeline** que desea ejecutar. La canalización tiene una única actividad que copia de una carpeta de origen en Azure Blob Storage a una carpeta de destino en el mismo almacenamiento. En la siguiente definición de JSON se muestra esta canalización de ejemplo:

```json
{
    "name": "copyPipeline",
    "properties": {
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },
                "name": "CopyBlobtoBlob",
                "inputs": [
                    {
                        "referenceName": "sourceBlobDataset",
                        "type": "DatasetReference"
                    }
                ],
                "outputs": [
                    {
                        "referenceName": "sinkBlobDataset",
                        "type": "DatasetReference"
                    }
                ]
            }
        ],
        "parameters": {
            "sourceBlobContainer": {
                "type": "String"
            },
            "sinkBlobContainer": {
                "type": "String"
            }
        }
    }
}
```

En la definición de JSON, la canalización toma dos parámetros: **sourceBlobContainer** y **sinkBlobContainer**. Los valores se pasan a estos parámetros en tiempo de ejecución.

Puede ejecutar manualmente la canalización mediante uno de los métodos siguientes:
- .NET SDK
- Módulo de Azure PowerShell
- API DE REST
- SDK de Python

### <a name="rest-api"></a>API DE REST

El comando de ejemplo siguiente muestra cómo ejecutar manualmente la canalización mediante la API REST:

```
POST
https://management.azure.com/subscriptions/mySubId/resourceGroups/myResourceGroup/providers/Microsoft.DataFactory/factories/myDataFactory/pipelines/copyPipeline/createRun?api-version=2017-03-01-preview
```

Para obtener un ejemplo completo, vea [Inicio rápido: Creación de una instancia de Azure Data Factory y una canalización mediante la API de REST](quickstart-create-data-factory-rest-api.md).

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

El comando de ejemplo siguiente muestra cómo ejecutar manualmente la canalización mediante Azure PowerShell:

```powershell
Invoke-AzDataFactoryV2Pipeline -DataFactory $df -PipelineName "Adfv2QuickStartPipeline" -ParameterFile .\PipelineParameters.json
```

Los parámetros se pasan en el cuerpo de la carga de solicitud. En el SDK de .NET, Azure PowerShell y el SDK de Python, los valores se pasan en un diccionario que se pasa como argumento a la llamada:

```json
{
  "sourceBlobContainer": "MySourceFolder",
  "sinkBlobContainer": "MySinkFolder"
}
```

La carga de respuesta es un identificador único de la ejecución de canalización:

```json
{
  "runId": "0448d45a-a0bd-23f3-90a5-bfeea9264aed"
}
```

Para obtener un ejemplo completo, vea [Inicio rápido: Creación de una factoría de datos de Azure PowerShell](quickstart-create-data-factory-powershell.md).

### <a name="net-sdk"></a>.NET SDK

La llamada de ejemplo siguiente muestra cómo ejecutar manualmente la canalización mediante el SDK de .NET:

```csharp
client.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName, parameters)
```

Para obtener un ejemplo completo, vea [Inicio rápido: Creación de una factoría de datos y una canalización con SDK de .NET](quickstart-create-data-factory-dot-net.md).

> [!NOTE]
> Puede utilizar el SDK de .NET para invocar las canalizaciones de Data Factory desde Azure Functions o de sus servicios web, entre otros.

## <a name="trigger-execution"></a>Ejecución del desencadenador

Los desencadenadores son otra forma de realizar una ejecución de canalización. Los desencadenadores representan una unidad de procesamiento que determina cuándo es necesario poner en marcha una ejecución de canalización. Actualmente, Data Factory admite tres tipos de desencadenadores:

- Desencadenador de programación: un desencadenador que invoca una canalización en una programación de reloj.

- Desencadenador de ventana de saltos de tamaño constante: un desencadenador que opera en un intervalo periódico, mientras conserva también el estado.

- Desencadenador basado en eventos: un desencadenador que responde a un evento.

Las canalizaciones y los desencadenadores tienen una relación de varios a varios (excepto el desencadenador de la ventana de saltos de tamaño constante). Varios desencadenadores pueden iniciar una sola canalización o un único desencadenador puede iniciar varias canalizaciones. En la siguiente definición de un desencadenador, la propiedad **pipelines** hace referencia a una lista de canalizaciones que desencadena un desencadenador en particular. La definición de propiedad incluye los valores para los parámetros de la canalización.
### <a name="basic-trigger-definition"></a>Definición básica del desencadenador

```json
{
    "properties": {
        "name": "MyTrigger",
        "type": "<type of trigger>",
        "typeProperties": {...},
        "pipelines": [
            {
                "pipelineReference": {
                    "type": "PipelineReference",
                    "referenceName": "<Name of your pipeline>"
                },
                "parameters": {
                    "<parameter 1 Name>": {
                        "type": "Expression",
                        "value": "<parameter 1 Value>"
                    },
                    "<parameter 2 Name>": "<parameter 2 Value>"
                }
            }
        ]
    }
}
```

## <a name="schedule-trigger"></a>Desencadenador de programación
Un desencadenador de programación ejecuta canalizaciones según una programación de reloj. Este desencadenador admite opciones de calendario periódicas y avanzadas. Por ejemplo, el desencadenador admite intervalos como "semanal" o "lunes a las 5:00 p. m. y jueves a las 9:00 p. m". El desencadenador de programación es flexible porque el patrón de conjunto de datos es independiente y el desencadenador no distingue entre datos de series temporales y datos de series no temporales.

Para más información sobre los desencadenadores de programación y ejemplos, vea [Creación de un desencadenador que ejecuta una canalización en una programación](how-to-create-schedule-trigger.md).

## <a name="schedule-trigger-definition"></a>Definición de desencadenador de programación
Cuando se crea un desencadenador de programación, debe especificar la programación y periodicidad mediante una definición de JSON.

Para hacer que el desencadenador de programación dé inicio a una ejecución de canalización, incluya una referencia de canalización de la canalización en particular en la definición del desencadenador. Las canalizaciones y los desencadenadores tienen una relación de varios a varios. Varios desencadenadores pueden comenzar una única canalización. Un único desencadenador puede iniciar varias canalizaciones.

```json
{
  "properties": {
    "type": "ScheduleTrigger",
    "typeProperties": {
      "recurrence": {
        "frequency": <<Minute, Hour, Day, Week, Year>>,
        "interval": <<int>>, // How often to fire
        "startTime": <<datetime>>,
        "endTime": <<datetime>>,
        "timeZone": "UTC",
        "schedule": { // Optional (advanced scheduling specifics)
          "hours": [<<0-24>>],
          "weekDays": [<<Monday-Sunday>>],
          "minutes": [<<0-60>>],
          "monthDays": [<<1-31>>],
          "monthlyOccurrences": [
            {
              "day": <<Monday-Sunday>>,
              "occurrence": <<1-5>>
            }
          ]
        }
      }
    },
  "pipelines": [
    {
      "pipelineReference": {
        "type": "PipelineReference",
        "referenceName": "<Name of your pipeline>"
      },
      "parameters": {
        "<parameter 1 Name>": {
          "type": "Expression",
          "value": "<parameter 1 Value>"
        },
        "<parameter 2 Name>": "<parameter 2 Value>"
      }
    }
  ]}
}
```

> [!IMPORTANT]
> La propiedad **parameters** es una propiedad obligatoria del elemento **pipelines**. Si la canalización no toma ningún parámetro, incluya una definición de JSON vacía para la propiedad **parameters**.

### <a name="schema-overview"></a>Información general del esquema
En la tabla siguiente se muestra información general de los elementos del esquema más importantes relacionados con la periodicidad y la programación de un desencadenador:

| Propiedad JSON | Descripción |
| --- | --- |
| **startTime** | Valor de fecha y hora. Para las programaciones básicas, se aplica el valor de la propiedad **startTime** al primer caso. Para las programaciones complejas, el desencadenador no se inicia antes del valor de **startTime** especificado. |
| **endTime** | Fecha y hora de finalización para el desencadenador. El desencadenador no se ejecuta después de la fecha y hora de finalización especificadas. El valor de la propiedad no puede estar en el pasado. <!-- This property is optional. --> |
| **timeZone** | La zona horaria Para obtener una lista de zonas horarias admitidas, vea [Creación de un desencadenador que ejecuta una canalización en una programación](how-to-create-schedule-trigger.md#time-zone-option). |
| **recurrence** | Objeto que especifica las reglas de periodicidad para el desencadenador. El objeto recurrence admite los elementos **frequency**, **interval**, **endTime**, **count** y **schedule**. Cuando se define un objeto recurrence, es necesario el elemento **frequency**. Los demás elementos del objeto recurrence son opcionales. |
| **frequency** | Unidad de frecuencia a la que se repite el desencadenador. Los valores admitidos son "minute", "hour", "day", "week" y "month". |
| **interval** | Entero positivo que indica el intervalo para el valor **frequency**. El valor **frequency** determina la frecuencia con la que se ejecuta el desencadenador. Por ejemplo, si **interval** es 3 y **frequency** es “week”, el desencadenador se repite cada tres semanas. |
| **schedule** | La programación de periodicidad para el desencadenador. Un desencadenador con valor de **frequency** especificado modifica su periodicidad según una programación periódica. La propiedad **schedule** contiene modificaciones de la periodicidad basadas en minutos, horas, días de la semana, días del mes y número de semana. |

### <a name="schedule-trigger-example"></a>Ejemplo de desencadenador de programación

```json
{
  "properties": {
    "name": "MyTrigger",
    "type": "ScheduleTrigger",
    "typeProperties": {
      "recurrence": {
        "frequency": "Hour",
        "interval": 1,
        "startTime": "2017-11-01T09:00:00-08:00",
        "endTime": "2017-11-02T22:00:00-08:00"
      }
    },
    "pipelines": [{
        "pipelineReference": {
          "type": "PipelineReference",
          "referenceName": "SQLServerToBlobPipeline"
        },
        "parameters": {}
      },
      {
        "pipelineReference": {
          "type": "PipelineReference",
          "referenceName": "SQLServerToAzureSQLPipeline"
        },
        "parameters": {}
      }
    ]
  }
}
```

### <a name="schema-defaults-limits-and-examples"></a>Valores predeterminados del esquema, límites y ejemplos

| Propiedad JSON | Tipo | Obligatorio | Valor predeterminado | Valores válidos | Ejemplo |
| --- | --- | --- | --- | --- | --- |
| **startTime** | string | Sí | None | Fechas y horas ISO-8601 | `"startTime" : "2013-01-09T09:30:00-08:00"` |
| **recurrence** | object | Sí | None | Objeto de periodicidad | `"recurrence" : { "frequency" : "monthly", "interval" : 1 }` |
| **interval** | number | No | 1 | 1 a 1000 | `"interval":10` |
| **endTime** | string | Sí | None | Valor de fecha y hora que representa un período de tiempo en el futuro | `"endTime" : "2013-02-09T09:30:00-08:00"` |
| **schedule** | object | No | None | Objeto de programación | `"schedule" : { "minute" : [30], "hour" : [8,17] }` |

### <a name="starttime-property"></a>Propiedad startTime
En la tabla siguiente se muestra cómo la propiedad **startTime** controla una ejecución de desencadenador:

| Valor de startTime | Periodicidad sin programación | Periodicidad con programación |
| --- | --- | --- |
| **La hora de inicio está en el pasado** | Calcula la primera hora de ejecución futura después de la hora de inicio y se ejecuta a esa hora.<br /><br />Realiza ejecuciones posteriores calculadas desde la última hora de ejecución.<br /><br />Consulte el ejemplo que sigue a esta tabla. | El desencadenador se inicia _no antes que_ la hora de inicio especificada. La primera repetición se basa en la programación que se calcula a partir de la hora de inicio.<br /><br />Realiza las sucesivas ejecuciones según la programación de periodicidad. |
| **La hora de inicio está en el futuro o la hora actual** | Se ejecuta una vez a la hora de inicio especificada.<br /><br />Realiza ejecuciones posteriores calculadas desde la última hora de ejecución. | El desencadenador se inicia _no antes_ que la hora de inicio especificada. La primera aparición se basa en la programación que se calcula a partir de la hora de inicio.<br /><br />Realiza las sucesivas ejecuciones según la programación de periodicidad. |

Veamos un ejemplo de lo que sucede cuando startTime se encuentra en el pasado, con periodicidad, pero sin programación. Suponga que la fecha actual es 2017-04-08 13:00, la hora de inicio es 2017-04-07 14:00, y la periodicidad es cada dos días. (El valor de **recurrence** se define estableciendo la propiedad **frequency** en "day" y la propiedad **interval** en 2). Tenga en cuenta que el valor de **startTime** se encuentra en el pasado y tiene lugar antes de la hora actual.

En estas condiciones, la primera ejecución será el 4 de septiembre de 2017 a las 14:00. El motor de Scheduler calcula las repeticiones de la ejecución desde la hora de inicio. Se descartan las instancias en el pasado. El motor utiliza la instancia siguiente que tiene lugar en el futuro. En este escenario, la hora de inicio es 2017-04-07 a las 2:00 p. m. La siguiente instancia es dos días a partir de ese momento, que es 2017-04-09 a las 2:00 p. m.

La hora de la primera ejecución es la misma si **startTime** es 2017-04-05 14:00 o 2017-04-01 14:00. Después de la primera ejecución, las ejecuciones posteriores se calculan mediante la programación. Por lo tanto, las ejecuciones siguientes se realizan el 2017-04-11 a las 2:00 p. m., luego el 2017-04-13 a las 2:00 p. m., después el 2017-04-15 a las 2:00 p. m. y así sucesivamente.

Finalmente, cuando las horas o los minutos no se establecen en el programa para un desencadenador, se utilizan las horas o minutos de la primera ejecución como valores predeterminados.

### <a name="schedule-property"></a>Propiedad schedule
Puede usar **schedule** para *limitar* el número de ejecuciones de desencadenadores. Por ejemplo, si un desencadenador con frecuencia mensual tiene un valor programado para ejecutarse solo el día 31, el desencadenador se ejecuta solo en los meses que tienen 31 días.

También puede usar **schedule** para *expandir* el número de ejecuciones de desencadenadores. Por ejemplo, un desencadenador con una frecuencia mensual programado para ejecutarse en los días primero y segundo del mes, se ejecuta el primer y segundo día del mes, en lugar de una vez al mes.

Si se especifican varios elementos de **programación**, el orden de evaluación es de la configuración de programación mayor a menor: número de semana, día del mes, día de la semana, hora y minuto.

En la siguiente tabla se describen los elementos de **schedule** con detalle:

| Elemento JSON | Descripción | Valores válidos |
| --- | --- | --- |
| **minutes** | Minutos de la hora en la que se ejecuta el desencadenador. |- Entero<br />- Matriz de enteros |
| **hours** | Horas del día en la que se ejecuta el desencadenador. |- Entero<br />- Matriz de enteros |
| **weekDays** | Días de la semana en los que se ejecuta el desencadenador. El valor solo se puede especificar con una frecuencia semanal.|<br />- Monday<br />- Tuesday<br />- Wednesday<br />- Thursday<br />- Friday<br />- Saturday<br />- Sunday<br />- Matriz de valores de día (el tamaño máximo de la matriz es 7)<br /><br />Los valores de día no distinguen mayúsculas de minúsculas. |
| **monthlyOccurrences** | Días del mes en los que se ejecuta el desencadenador. El valor solo se puede especificar con una frecuencia mensual. |- Matriz de objetos de **monthlyOccurrence**: `{ "day": day, "occurrence": occurrence }`.<br />- El atributo **day** es el día de la semana en el que se ejecuta el desencadenador. Por ejemplo, una propiedad **monthlyOccurrences** con un valor de **day** de `{Sunday}` significa todos los domingos del mes. Se necesita un atributo **day**.<br />- El atributo **occurrence** es la repetición del elemento **day** especificado durante el mes. Por ejemplo, una propiedad **monthlyOccurrences** valores de **day** y **occurrence** de `{Sunday, -1}` implica el último domingo del mes. El atributo **occurrence** es opcional. |
| **monthDays** | Día del mes en el que se ejecuta el desencadenador. El valor solo se puede especificar con una frecuencia mensual. |- Cualquier valor <= -1 y >= -31<br />- Cualquier valor >= 1 y <= 31<br />- Matriz de valores |

## <a name="tumbling-window-trigger"></a>Desencadenador de ventana de saltos de tamaño constante
Los desencadenadores de ventana de saltos de tamaño constante son un tipo de desencadenador que se activa en un intervalo de tiempo periódico a partir de una hora de inicio especificada, mientras conserva el estado. Las ventanas de saltos de tamaño constante son una serie de intervalos de tiempo de tamaño fijo, contiguos y que no se superponen.

Para más información sobre los desencadenadores de ventanas de saltos de tamaño constante y ejemplos, consulte [Creación de un desencadenador de ventana de saltos de tamaño constante](how-to-create-tumbling-window-trigger.md).

## <a name="examples-of-trigger-recurrence-schedules"></a>Ejemplos de programaciones de periodicidad del desencadenador

En esta sección se proporcionan ejemplos de programaciones de periodicidad. Se centra en el objeto **schedule** y sus elementos.

Los ejemplos asumen que el valor de **interval** es 1 y que el valor de **frequency** es correcto según la definición de la programación. Por ejemplo, no puede tener un valor de **frequency** de "day" y tener también una modificación de **monthDays** en el objeto **schedule**. Estos tipos de restricciones se describen en la tabla de la sección anterior.

| Ejemplo | Descripción |
| --- | --- |
| `{"hours":[5]}` | Se ejecuta a las 5:00 a. m. todos los días. |
| `{"minutes":[15], "hours":[5]}` | Se ejecuta a las 5:15 a. m. todos los días. |
| `{"minutes":[15], "hours":[5,17]}` | Se ejecuta a las 5:15 a. m. y 5:15 p. m. todos los días. |
| `{"minutes":[15,45], "hours":[5,17]}` | Se ejecuta a las 5:15 a. m., 5:45 a. m., 5:15 p. m. y 5:45 p. m. todos los días. |
| `{"minutes":[0,15,30,45]}` | Se ejecuta cada 15 minutos. |
| `{hours":[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23]}` | Se ejecuta cada hora.<br /><br />Este desencadenador se ejecuta cada hora. Los minutos se controlan mediante el valor de **startTime**, cuando se especifica un valor. Si no se especifica un valor, los minutos los controla el tiempo de creación. Por ejemplo, si la hora de inicio o la hora de creación (lo que corresponda) es 12:25 p. m., el desencadenador se ejecuta a las 00:25, 01:25, 02:25… y 23:25.<br /><br />Su programación equivale a tener un desencadenador con un valor de **frequency** de "hour", un valor de **interval** de 1 y ningún valor de **schedule**. Esta programación puede usarse con diferentes valores en **frequency** e **interval** para crear otros desencadenadores. Por ejemplo, cuando el valor de **frequency** es "month", la ejecución de la programación se ejecuta solo una vez al mes, en lugar de cada día, cuando el valor de **frequency** es "day". |
| `{"minutes":[0]}` | Se ejecuta cada hora durante la hora.<br /><br />Este desencadenador se ejecuta cada hora a partir de las 12:00 a. m., 1:00 a. m., 2:00 a. m., y así sucesivamente.<br /><br />Esta programación es equivalente a un desencadenador con un valor de **frequency** de "hour" y un valor de **startTime** de cero minutos, y sin valor de **schedule** pero con un valor de **frequency** de "day". Si el valor de **frequency** es "week" o "month", la programación ejecuta únicamente un día a la semana o un día al mes, respectivamente. |
| `{"minutes":[15]}` | Se ejecuta 15 minutos después de cada hora en punto.<br /><br />Este desencadenador se ejecuta cada hora 15 minutos después de la hora en punto desde las 00:15 a. m., 1:15 a. m., 2:15 a. m. y así sucesivamente, y finalizando a las 11:15 p. m. |
| `{"hours":[17], "weekDays":["saturday"]}` | Se ejecuta a las 5:00 p. m. los sábados de cada semana. |
| `{"hours":[17], "weekDays":["monday", "wednesday", "friday"]}` | Se ejecuta a las 5:00 p. m. los lunes, miércoles y viernes de cada semana. |
| `{"minutes":[15,45], "hours":[17], "weekDays":["monday", "wednesday", "friday"]}` | Se ejecuta a las 5:15 p. m. y 5:45 p. m. los lunes, miércoles y viernes de cada semana. |
| `{"minutes":[0,15,30,45], "weekDays":["monday", "tuesday", "wednesday", "thursday", "friday"]}` | Se ejecuta cada 15 minutos los días laborables. |
| `{"minutes":[0,15,30,45], "hours": [9, 10, 11, 12, 13, 14, 15, 16] "weekDays":["monday", "tuesday", "wednesday", "thursday", "friday"]}` | Se ejecuta cada 15 minutos los días laborables entre las 9:00 a. m. y las 4:45 p. m. |
| `{"weekDays":["tuesday", "thursday"]}` | Se ejecuta los martes y jueves a la hora de inicio especificada. |
| `{"minutes":[0], "hours":[6], "monthDays":[28]}` | Se ejecuta a las 6:00 a. m. del día 28 de cada mes (suponiendo un valor de **frequency** de "month"). |
| `{"minutes":[0], "hours":[6], "monthDays":[-1]}` | Se ejecuta a las 6:00 a. m. el último día del mes.<br /><br />Para ejecutar un desencadenador el último día del mes, use -1 en lugar del día 28, 29, 30 o 31. |
| `{"minutes":[0], "hours":[6], "monthDays":[1,-1]}` | Se ejecuta a las 6:00 a. m. el primer y el último día del mes. |
| `{monthDays":[1,14]}` | Se ejecuta el primer y decimocuarto día de cada mes a la hora de inicio especificada. |
| `{"minutes":[0], "hours":[5], "monthlyOccurrences":[{"day":"friday", "occurrence":1}]}` | Se ejecuta el primer viernes de cada mes a las 5:00 a. m. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":1}]}` | Se ejecuta el primer viernes de cada mes a la hora de inicio especificada. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":-3}]}` | Se ejecuta el tercer viernes desde el final del mes, todos los meses, a la hora de inicio especificada. |
| `{"minutes":[15], "hours":[5], "monthlyOccurrences":[{"day":"friday", "occurrence":1},{"day":"friday", "occurrence":-1}]}` | Se ejecuta el primer viernes de cada mes a las 5:15 a. m. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":1},{"day":"friday", "occurrence":-1}]}` | Se ejecuta el primer y el último viernes de cada mes a la hora de inicio especificada. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":5}]}` | Se ejecuta el quinto viernes de cada mes a la hora de inicio especificada.<br /><br />Si no existe el quinto viernes de un mes, la canalización no se ejecuta. Considere usar -1 en lugar de 5 para el valor **occurrence** para ejecutar el desencadenador en el último viernes del mes. |
| `{"minutes":[0,15,30,45], "monthlyOccurrences":[{"day":"friday", "occurrence":-1}]}` | Se ejecuta cada 15 minutos el último viernes del mes. |
| `{"minutes":[15,45], "hours":[5,17], "monthlyOccurrences":[{"day":"wednesday", "occurrence":3}]}` | Se ejecuta a las 5:15 a. m., 5:45 a. m., 5:15 a. m. y las 5:45 a. m. el tercer miércoles de cada mes. |

## <a name="trigger-type-comparison"></a>Comparación de tipos de desencadenadores

El desencadenador de ventana de saltos de tamaño constante y el de programación funcionan con latidos de tiempo. ¿En qué se diferencian?

> [!NOTE]
> El desencadenador de la ventana de saltos *espera a que se ejecute la canalización desencadenada* para finalizar. Su estado de ejecución refleja el estado de la ejecución de la canalización desencadenada. Por ejemplo, si se cancela una ejecución de canalización desencadenada, la ejecución del desencadenador de la ventana de saltos correspondiente se marca como cancelada. Esto es diferente del comportamiento "desencadenar y olvidar" del desencadenador de programación, que se marca como correcto siempre que se inicie una ejecución de canalización.

En la siguiente tabla se muestra una comparación entre el desencadenador de ventana de saltos de tamaño constante y el desencadenador de programación:

| Elemento | Desencadenador de ventana de saltos de tamaño constante | Desencadenador de programación |
| --- | --- | --- |
| **Escenarios de reposición** | Compatible. Las ejecuciones de canalización se pueden programar para ventanas en el pasado. | No compatible. Se pueden realizar las ejecuciones de canalización solo en períodos de tiempo del momento actual y en el futuro. |
| **Confiabilidad** | 100 % confiabilidad. Las ejecuciones de canalización se pueden programar para todas las ventanas de una fecha de inicio especificada sin intervalos. | Menos confiable. |
| **Funcionalidad de reintento** | Compatible. Las ejecuciones de canalización erróneas tienen una directiva de reintentos predeterminada de 0 u otra especificada por el usuario en la definición del desencadenador. Realiza un reintento automáticamente cuando se produce un error en la ejecución de la canalización debido a los límites de simultaneidad/servidor/limitación (es decir, códigos de estado 400: Error de usuario, 429: Demasiadas solicitudes y 500: Error interno del servidor). | No compatible. |
| **Concurrency** | Compatible. Los usuarios pueden establecer explícitamente límites de simultaneidad para el desencadenador. Permite entre 1 y 50 ejecuciones simultáneas de canalizaciones desencadenadas. | No compatible. |
| **Variables del sistema** | Junto con @trigger().scheduledTime y @trigger().startTime, también admite el uso de las variables del sistema **WindowStart** y **WindowEnd**. Los usuarios pueden acceder a `trigger().outputs.windowStartTime` y `trigger().outputs.windowEndTime` como variables del sistema del desencadenador en la definición del desencadenador. Los valores se utilizan como la hora de inicio y la hora de finalización de la ventana, respectivamente. Por ejemplo, para un desencadenador de la ventana de saltos de tamaño constante que se ejecuta cada hora, para la ventana de 1:00 a. m. a 2:00 a. m., la definición es `trigger().outputs.windowStartTime = 2017-09-01T01:00:00Z` y `trigger().outputs.windowEndTime = 2017-09-01T02:00:00Z`. | Solo admite las variables predeterminadas @trigger().scheduledTime y @trigger().startTime. |
| **Relación de canalización a desencadenador** | Admite las relaciones uno a uno. Solo se puede desencadenar una canalización. | Admite relaciones muchos a muchos. Varios desencadenadores pueden comenzar una única canalización. Un único desencadenador puede iniciar varias canalizaciones. |

## <a name="event-based-trigger"></a>Desencadenador basado en eventos

Un desencadenador basado en eventos ejecuta canalizaciones en respuesta a un evento. Hay dos tipos de desencadenadores basados en eventos.

* El _desencadenador de eventos de almacenamiento_ ejecuta una canalización con los eventos que ocurren en una cuenta de almacenamiento, como la llegada de un archivo o la eliminación de un archivo en la cuenta de Azure Blob Storage.
* El _desencadenador de eventos personalizados_ procesa y administra [temas personalizados](../event-grid/custom-topics.md) en Event Grid

Para obtener más información acerca de los desencadenadores basados en eventos, vea [desencadenador de eventos de almacenamiento](how-to-create-event-trigger.md) y [desencadenador de eventos personalizados](how-to-create-custom-event-trigger.md).

## <a name="next-steps"></a>Pasos siguientes

Vea los siguientes tutoriales:

- [Inicio rápido: Creación de una factoría de datos y una canalización con SDK de .NET](quickstart-create-data-factory-dot-net.md).
- [Creación de un desencadenador que ejecuta una canalización en una programación](how-to-create-schedule-trigger.md)
- [Creación de un desencadenador que ejecuta una canalización en una ventana de saltos de tamaño constante](how-to-create-tumbling-window-trigger.md)
