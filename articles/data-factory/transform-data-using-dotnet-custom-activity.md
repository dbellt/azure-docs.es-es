---
title: Uso de actividades personalizadas en una canalización
description: Aprenda a crear actividades personalizadas mediante .NET y, luego, úselas en una canalización de Azure Data Factory.
ms.service: data-factory
author: nabhishek
ms.author: abnarain
ms.topic: conceptual
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.date: 11/26/2018
ms.openlocfilehash: 3b5370baacc2bf82ae0575d44d00d1535a4549de
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110665441"
---
# <a name="use-custom-activities-in-an-azure-data-factory-pipeline"></a>Uso de actividades personalizadas en una canalización de Azure Data Factory

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-use-custom-activities.md)
> * [Versión actual](transform-data-using-dotnet-custom-activity.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Hay dos tipos de actividades que puede usar en una canalización de Azure Data Factory.

- [Actividades de movimiento de datos](copy-activity-overview.md) para mover datos entre [almacenes de datos de origen y receptor compatibles](copy-activity-overview.md#supported-data-stores-and-formats).
- [Actividades de transformación de datos](transform-data.md) para transformar datos mediante procesos como Azure HDInsight, Azure Batch y Azure Machine Learning.

Para mover datos desde y hacia un almacén de datos incompatible con Data Factory, o para transformar o procesar datos de algún modo incompatible con Data Factory, puede crear una **actividad personalizada** con su propia lógica de desplazamiento o transformación de datos y usarla en una canalización. La actividad personalizada ejecuta la lógica del código personalizado en un grupo de máquinas virtuales de **Azure Batch**.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Consulte los artículos siguientes si no está familiarizado con el servicio Azure Batch:

* [Aspectos básicos de Azure Batch](../batch/batch-technical-overview.md) para información general del servicio Azure Batch.
* Cmdlet [New-AzBatchAccount](/powershell/module/az.batch/New-azBatchAccount) para crear una cuenta de Azure Batch, o [Azure Portal](../batch/batch-account-create-portal.md) para crear la cuenta de Azure Batch con Azure Portal. Consulte el artículo [Using PowerShell to manage Azure Batch Account](/archive/blogs/windowshpc/using-azure-powershell-to-manage-azure-batch-account) (Administración de cuentas de Azure Batch con PowerShell) para instrucciones detalladas sobre el uso del cmdlet.
* [New-AzBatchPool](/powershell/module/az.batch/New-AzBatchPool) para crear un grupo de Azure Batch.

> [!IMPORTANT]
> Al crear un grupo de Azure Batch nuevo, se debe usar "VirtualMachineConfiguration", NO "CloudServiceConfiguration". Para más información, consulte la [guía de migración de grupos de Azure Batch](../batch/batch-pool-cloud-service-to-virtual-machine-configuration.md). 

## <a name="azure-batch-linked-service"></a>Servicio vinculado de Azure Batch

El siguiente JSON define un servicio vinculado de Azure Batch de ejemplo. Para obtener información detallada, vea [Compute environments supported by Azure Data Factory](compute-linked-services.md) (Entornos de proceso compatibles con Azure Data Factory).

```json
{
    "name": "AzureBatchLinkedService",
    "properties": {
        "type": "AzureBatch",
        "typeProperties": {
            "accountName": "batchaccount",
            "accessKey": {
                "type": "SecureString",
                "value": "access key"
            },
            "batchUri": "https://batchaccount.region.batch.azure.com",
            "poolName": "poolname",
            "linkedServiceName": {
                "referenceName": "StorageLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }
}
```

 Para más información sobre el servicio vinculado de Azure Batch, vea el artículo [Servicios vinculados de proceso](compute-linked-services.md).

## <a name="custom-activity"></a>Actividad personalizada

El siguiente fragmento de código JSON define una canalización con una actividad personalizada simple. La definición de actividad tiene una referencia al servicio vinculado de Azure Batch.

```json
{
  "name": "MyCustomActivityPipeline",
  "properties": {
    "description": "Custom activity sample",
    "activities": [{
      "type": "Custom",
      "name": "MyCustomActivity",
      "linkedServiceName": {
        "referenceName": "AzureBatchLinkedService",
        "type": "LinkedServiceReference"
      },
      "typeProperties": {
        "command": "helloworld.exe",
        "folderPath": "customactv2/helloworld",
        "resourceLinkedService": {
          "referenceName": "StorageLinkedService",
          "type": "LinkedServiceReference"
        }
      }
    }]
  }
}
```

En este ejemplo, el archivo helloworld.exe es una aplicación personalizada que se almacena en la carpeta customactv2/helloworld de la cuenta de Azure Storage usada en la propiedad resourceLinkedService. La actividad personalizada envía esta aplicación personalizada para ejecutarla en Azure Batch. Puede reemplazar el comando en cualquier aplicación preferida que se pueda ejecutar en el sistema operativo de destino de los nodos del grupo de Azure Batch.

En la tabla siguiente se describen los nombres y descripciones de las propiedades que son específicas de esta actividad.

| Propiedad              | Descripción                              | Obligatorio |
| :-------------------- | :--------------------------------------- | :------- |
| name                  | Nombre de la actividad en la canalización     | Sí      |
| description           | Texto que describe para qué se usa la actividad.  | No       |
| type                  | Para la actividad personalizada, el tipo de actividad es **Custom**. | Sí      |
| linkedServiceName     | Servicio vinculado a Azure Batch. Para obtener más información sobre este servicio vinculado, vea el artículo [Compute linked services](compute-linked-services.md) (Servicios vinculados de procesos).  | Sí      |
| command               | Comando de la aplicación personalizada que se va a ejecutar. Si la aplicación ya está disponible en el nodo del grupo de Azure Batch, se pueden omitir las propiedades resourceLinkedService y folderPath. Por ejemplo, puede especificar que el comando sea `cmd /c dir`, que el nodo del grupo de lotes de Windows admite de forma nativa. | Sí      |
| resourceLinkedService | Servicio de Azure Storage vinculado a la cuenta de almacenamiento en la que está almacenada la aplicación personalizada | No &#42;       |
| folderPath            | Ruta de acceso a la carpeta de la aplicación personalizada y todas sus dependencias<br/><br/>Si tiene dependencias que se almacenan en subcarpetas (es decir, en una estructura jerárquica de carpetas bajo *folderPath*) la estructura de carpetas se elimina cuando los archivos se copian en Azure Batch. Es decir, todos los archivos se copian en una sola carpeta sin subcarpetas. Para evitar este comportamiento, considere la posibilidad de comprimir los archivos, copiar el archivo comprimido y, a continuación, descomprimirlo con código personalizado en la ubicación deseada. | No &#42;       |
| referenceObjects      | Matriz de servicios vinculados y conjuntos de datos existentes. Los servicios vinculados y los conjuntos de datos a los que se hace referencia se pasan a la aplicación personalizada en formato JSON, por lo que el código personalizado puede hacer referencia a recursos de Data Factory | No       |
| extendedProperties    | Propiedades definidas por el usuario que se pueden pasar a la aplicación personalizada en formato JSON, por lo que el código personalizado puede hacer referencia a propiedades adicionales | No       |
| retentionTimeInDays | Tiempo de retención de los archivos enviados para la actividad personalizada. El valor predeterminado es 30 días. | No |

&#42;Las propiedades `resourceLinkedService` y `folderPath` deben especificarse ambas u omitirse ambas.

> [!NOTE]
> Si se pasan servicios vinculados como referenceObjects en la actividad personalizada, una buena práctica de seguridad consiste en pasar un servicio vinculado habilitado para Azure Key Vault (ya que no contiene cadenas seguras) y obtener las credenciales usando el nombre del secreto directamente de Key Vault desde el código. [Aquí](https://github.com/nabhishek/customactivity_sample/tree/linkedservice) encontrará un ejemplo donde se hace referencia a un servicio vinculado habilitado para AKV, se recuperan las credenciales de Key Vault y, después, se accede al almacenamiento en el código.

## <a name="custom-activity-permissions"></a>Permisos de la actividad personalizada

La actividad personalizada establece la cuenta de usuario automático de Azure Batch en *Acceso sin privilegios de administrador con ámbito de la tarea* (la especificación de usuario automático predeterminada). No se puede cambiar el nivel de permiso de la cuenta de usuario automático. Para más información, consulte [Ejecución de tareas en cuentas de usuario en Batch | Cuentas de usuario automático](../batch/batch-user-accounts.md#auto-user-accounts).

## <a name="executing-commands"></a>Ejecutar comandos

Puede ejecutar directamente un comando mediante la actividad personalizada. En el ejemplo siguiente se ejecuta un comando "echo hello world" en los nodos de grupo de destino de Azure Batch y se imprime la salida en stdout.

```json
{
  "name": "MyCustomActivity",
  "properties": {
    "description": "Custom activity sample",
    "activities": [{
      "type": "Custom",
      "name": "MyCustomActivity",
      "linkedServiceName": {
        "referenceName": "AzureBatchLinkedService",
        "type": "LinkedServiceReference"
      },
      "typeProperties": {
        "command": "cmd /c echo hello world"
      }
    }]
  }
}
```

## <a name="passing-objects-and-properties"></a>Pasar objetos y propiedades

En este ejemplo se muestra cómo usar las propiedades referenceObjects y extendedProperties para pasar objetos de Data Factory y propiedades definidas por el usuario a la aplicación personalizada.

```json
{
  "name": "MyCustomActivityPipeline",
  "properties": {
    "description": "Custom activity sample",
    "activities": [{
      "type": "Custom",
      "name": "MyCustomActivity",
      "linkedServiceName": {
        "referenceName": "AzureBatchLinkedService",
        "type": "LinkedServiceReference"
      },
      "typeProperties": {
        "command": "SampleApp.exe",
        "folderPath": "customactv2/SampleApp",
        "resourceLinkedService": {
          "referenceName": "StorageLinkedService",
          "type": "LinkedServiceReference"
        },
        "referenceObjects": {
          "linkedServices": [{
            "referenceName": "AzureBatchLinkedService",
            "type": "LinkedServiceReference"
          }]
        },
        "extendedProperties": {          
          "connectionString": {
            "type": "SecureString",
            "value": "aSampleSecureString"
          },
          "PropertyBagPropertyName1": "PropertyBagValue1",
          "propertyBagPropertyName2": "PropertyBagValue2",
          "dateTime1": "2015-04-12T12:13:14Z"
        }
      }
    }]
  }
}
```

Al ejecutar la actividad, las propiedades referenceObjects y extendedProperties se almacenan en los siguientes archivos que se implementan en la misma carpeta de ejecución del archivo SampleApp.exe:

- `activity.json`

  Almacena extendedProperties y las propiedades de la actividad personalizada.

- `linkedServices.json`

  Almacena una matriz de servicios vinculados definidos en la propiedad referenceObjects.

- `datasets.json`

  Almacena una matriz de conjuntos de datos definidos en la propiedad referenceObjects.

En el siguiente código de ejemplo se muestra cómo SampleApp.exe puede obtener acceso a la información necesaria de los archivos JSON:

```csharp
using Newtonsoft.Json;
using System;
using System.IO;

namespace SampleApp
{
    class Program
    {
        static void Main(string[] args)
        {
            //From Extend Properties
            dynamic activity = JsonConvert.DeserializeObject(File.ReadAllText("activity.json"));
            Console.WriteLine(activity.typeProperties.extendedProperties.connectionString.value);

            // From LinkedServices
            dynamic linkedServices = JsonConvert.DeserializeObject(File.ReadAllText("linkedServices.json"));
            Console.WriteLine(linkedServices[0].properties.typeProperties.accountName);
        }
    }
}
```

## <a name="retrieve-execution-outputs"></a>Recuperar los resultados de la ejecución

Puede iniciar una ejecución de canalización mediante el siguiente comando de PowerShell:

```powershell
$runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName $pipelineName
```

Durante la ejecución de la canalización, puede comprobar la salida de la ejecución mediante los comandos siguientes:

```powershell
while ($True) {
    $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)

    if(!$result) {
        Write-Host "Waiting for pipeline to start..." -foregroundcolor "Yellow"
    }
    elseif (($result | Where-Object { $_.Status -eq "InProgress" } | Measure-Object).count -ne 0) {
        Write-Host "Pipeline run status: In Progress" -foregroundcolor "Yellow"
    }
    else {
        Write-Host "Pipeline '"$pipelineName"' run finished. Result:" -foregroundcolor "Yellow"
        $result
        break
    }
    ($result | Format-List | Out-String)
    Start-Sleep -Seconds 15
}

Write-Host "Activity `Output` section:" -foregroundcolor "Yellow"
$result.Output -join "`r`n"

Write-Host "Activity `Error` section:" -foregroundcolor "Yellow"
$result.Error -join "`r`n"
```

Los archivos **stdout** y **stderr** de la aplicación personalizada se guardan en el contenedor **adfjobs** del servicio vinculado de Azure Storage definido al crear el servicio vinculado de Azure Batch con un GUID de la tarea. Puede obtener la ruta de acceso detallada de la salida de la ejecución de la actividad tal como se muestra en el siguiente fragmento de código:

```
Pipeline ' MyCustomActivity' run finished. Result:

ResourceGroupName : resourcegroupname
DataFactoryName   : datafactoryname
ActivityName      : MyCustomActivity
PipelineRunId     : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
PipelineName      : MyCustomActivity
Input             : {command}
Output            : {exitcode, outputs, effectiveIntegrationRuntime}
LinkedServiceName :
ActivityRunStart  : 10/5/2017 3:33:06 PM
ActivityRunEnd    : 10/5/2017 3:33:28 PM
DurationInMs      : 21203
Status            : Succeeded
Error             : {errorCode, message, failureType, target}

Activity Output section:
"exitcode": 0
"outputs": [
  "https://<container>.blob.core.windows.net/adfjobs/<GUID>/output/stdout.txt",
  "https://<container>.blob.core.windows.net/adfjobs/<GUID>/output/stderr.txt"
]
"effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)"
Activity Error section:
"errorCode": ""
"message": ""
"failureType": ""
"target": "MyCustomActivity"
```

Si desea usar el contenido de stdout.txt en actividades de bajada, puede obtener la ruta al archivo stdout.txt en la expresión "\@activity('MyCustomActivity').output.outputs[0]".

> [!IMPORTANT]
> - Los archivos activity.json, linkedServices.json y datasets.json se almacenan en la carpeta de tiempo de ejecución de la tarea de Batch. En este ejemplo, los archivos activity.json, linkedServices.json y datasets.json se almacenan en la ruta de acceso `https://adfv2storage.blob.core.windows.net/adfjobs/<GUID>/runtime/`. Si es necesario, deberá limpiarlos por separado.
> - En cuanto a los servicios vinculados que usan un entorno de ejecución de integración autohospedado, la información confidencial (como claves o contraseñas) se cifra mediante el entorno de ejecución de integración autohospedado para garantizar que las credenciales permanecen en el entorno de red privada que haya definido el cliente. Si el código de la aplicación personalizada hace referencia de esta forma a algunos campos confidenciales, es posible que estos no estén presentes. Si es necesario, use SecureString en extendedProperties en lugar de usar la referencia al servicio vinculado.

## <a name="pass-outputs-to-another-activity"></a>Resultados del paso a otra actividad

Puede enviar los valores personalizados desde el código de una actividad personalizada a Azure Data Factory. Puede hacerlo escribiéndolos en `outputs.json` desde la aplicación. Data Factory copia el contenido de `outputs.json` y lo anexa a la salida de la actividad como el valor de la propiedad `customOutput`. (El límite de tamaño es de 2 MB). Si quiere consumir el contenido de `outputs.json` en las actividades posteriores, puede obtener el valor mediante la expresión `@activity('<MyCustomActivity>').output.customOutput`.

## <a name="retrieve-securestring-outputs"></a>Recuperación de salidas de SecureString

Los valores de propiedades confidenciales designados como de tipo *SecureString*, tal y como se muestra en algunos de los ejemplos de este artículo, se enmascaran en la pestaña Supervisión de la interfaz de usuario de Data Factory.  Sin embargo, en la ejecución de la canalización real, una propiedad *SecureString* se serializa como JSON dentro del archivo `activity.json` como texto sin formato. Por ejemplo:

```json
"extendedProperties": {
  "connectionString": {
    "type": "SecureString",
    "value": "aSampleSecureString"
  }
}
```

Esta serialización no es verdaderamente segura y no está pensada para serlo. La intención es sugerir que Data Factory enmascare el valor en la pestaña Supervisión.

Para acceder a las propiedades de tipo *SecureString* desde una actividad personalizada, lea el archivo `activity.json`, que se coloca en la misma carpeta que su archivo .EXE, deserialice el archivo JSON y, a continuación, acceda a la propiedad JSON (extendedProperties => [propertyName] => valor).

## <a name="compare-v2-custom-activity-and-version-1-custom-dotnet-activity"></a><a name="compare-v2-v1"></a> Comparación de la actividad personalizada de la versión 2 y la actividad de DotNet de la versión 1 (personalizada)

En la versión 1 de Azure Data Factory, puede implementar una actividad de DotNet (personalizada) mediante la creación de un proyecto de la biblioteca de clases .NET con una clase que implementa el método `Execute` de la interfaz `IDotNetActivity`. Los servicios vinculados, los conjuntos de datos y las propiedades extendidas de la carga de JSON de una actividad de DotNet (personalizada) se pasan al método de ejecución como objetos fuertemente tipados. Para detalles sobre el comportamiento de la versión 1, consulte la [actividad de DotNet (personalizada) en la versión 1](v1/data-factory-use-custom-activities.md). Debido a esta implementación, el código de la actividad de DotNet de la versión 1 tiene como destino .NET Framework 4.5.2. La actividad de DotNet de la versión 1 también se debe ejecutar en nodos de grupo de Azure Batch basados en Windows.

En la actividad personalizada de Azure Data Factory V2, no es necesario que implemente ninguna interfaz .NET. Ahora puede ejecutar directamente comandos, scripts y su propio código personalizado, compilado como ejecutable. Para configurar esta implementación, debe especificar la propiedad `Command` en conjunto con la propiedad `folderPath`. La actividad personalizada carga el ejecutable y sus dependencias en `folderpath` y ejecuta el comando en su lugar.

El ejecutable puede acceder a los servicios vinculados, a los conjuntos de datos (definidos en referenceObjects) y a las propiedades extendidas definidas en la carga de JSON de una actividad personalizada de Data Factory v2. Puede acceder a las propiedades requeridas mediante un serializador JSON como se muestra en el ejemplo de código SampleApp.exe anterior.

Con los cambios introducidos en la actividad personalizada de Data Factory V2, puede escribir su lógica de código personalizado en el lenguaje que desee y ejecutarlo en sistemas operativos Windows o Linux que sean compatibles con Azure Batch.

En la tabla siguiente se describen las diferencias entre la actividad personalizada de la versión 2 de Data Factory y la actividad de DotNet (personalizada) de la versión 1 de Data Factory:

|Diferencias      | Actividad personalizada      | Actividad de DotNet (personalizada) de la versión 1      |
| ---- | ---- | ---- |
|Formas de definir la lógica personalizada      |Proporcionar un ejecutable      |Implementar un archivo DLL de .Net      |
|Entorno de ejecución de la lógica personalizada      |Windows o Linux      |Windows (.NET Framework 4.5.2)      |
|Ejecución de scripts      |Admite la ejecución directa de scripts (por ejemplo "cmd /c echo hello world" en la VM Windows)      |Requiere la implementación en el archivo DLL de .NET      |
|Conjunto de datos requerido      |Opcional      |Necesario para encadenar actividades y pasar información      |
|Pasar información de actividad a la lógica personalizada      |Mediante ReferenceObjects (LinkedServices y conjuntos de datos) y ExtendedProperties (propiedades personalizadas)      |Mediante ExtendedProperties (propiedades personalizadas), y conjuntos de datos de entrada y salida      |
|Recuperación de información en lógica personalizada      |Analiza los archivos activity.json, linkedServices.json y datasets.json almacenados en la misma carpeta que el ejecutable      |Mediante SDK de .NET (.NET Frame 4.5.2)      |
|Registro      |Escribe directamente en STDOUT      |Se implementa el registrador en el archivo DLL de .NET      |

Si tiene código .NET ya existente escrito para la actividad de DotNet (personalizada) de la versión 1, deberá modificar el código para que funcione con la versión actual de la actividad personalizada. Siga estas directrices de alto nivel para actualizar el código:

  - Cambie el proyecto de una biblioteca de clases .NET a una aplicación de consola.
  - Inicie la aplicación con el método `Main`. El método `Execute` de la interfaz `IDotNetActivity` ya no es necesario.
  - Lea y analice los servicios vinculados, los conjuntos de datos y la actividad con un serializador JSON y como objetos fuertemente tipados. Pase los valores de las propiedades requeridas a la lógica del código personalizado principal. Consulte el código SampleApp.exe anterior como ejemplo.
  - Ya no se admite el objeto del registrador. La salida del ejecutable se puede imprimir en la consola y se guarda en stdout.txt.
  - Ya no es necesario el paquete NuGet Microsoft.Azure.Management.DataFactories.
  - Compile el código, cargue el ejecutable y sus dependencias en Azure Storage y defina la ruta de acceso en la propiedad `folderPath`.

Para un ejemplo completo de cómo el archivo DLL entero y el ejemplo de canalización que se describen en el artículo [Uso de actividades personalizadas en una canalización de Azure Data Factory](./v1/data-factory-use-custom-activities.md) de la versión 1 de Data Factory se pueden reescribir como una actividad personalizada de Data Factory, consulte un [ejemplo de la actividad personalizada de Data Factory](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/ADFv2CustomActivitySample).

## <a name="auto-scaling-of-azure-batch"></a>Escalado automático de Azure Batch

También puede crear un grupo de Azure Batch con la característica **autoescala** . Por ejemplo, podría crear un grupo de Azure Batch con 0 VM dedicadas y una fórmula de escalado automático basada en el número de tareas pendientes.

La fórmula del ejemplo obtiene el comportamiento siguiente: Cuando el grupo se crea inicialmente, se inicia con una máquina virtual. La métrica $PendingTasks define el número de tareas que están en ejecución o activas (en cola). La fórmula busca el número promedio de tareas pendientes en los últimos 180 segundos y establece TargetDedicated en consecuencia. Garantiza que TargetDedicated nunca supera las 25 VM. Por tanto, a medida que se envían nuevas tareas, el grupo crece automáticamente y a medida que estás se completan, las VM se liberan una a una y el escalado automático las reduce. startingNumberOfVMs y maxNumberofVMs se pueden adaptar a sus necesidades.

Fórmula de escalado automático:

```
startingNumberOfVMs = 1;
maxNumberofVMs = 25;
pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
$TargetDedicated=min(maxNumberofVMs,pendingTaskSamples);
```

Para más información, consulte [Escalado automático de los nodos de proceso en un grupo de Azure Batch](../batch/batch-automatic-scaling.md) .

Si el grupo usa el valor predeterminado de la propiedad [autoScaleEvaluationInterval](/rest/api/batchservice/pool/enableautoscale), el servicio Batch puede tardar de 15 a 30 minutos en preparar la máquina virtual antes de ejecutar la actividad personalizada. Si el grupo usa otro valor de autoScaleEvaluationInterval diferente, el servicio Batch podría tardar el valor de autoScaleEvaluationInterval más 10 minutos.

## <a name="next-steps"></a>Pasos siguientes
Vea los siguientes artículos, en los que se explica cómo transformar datos de otras maneras:

* [Actividad de U-SQL](transform-data-using-data-lake-analytics.md)
* [Actividad de Hive](transform-data-using-hadoop-hive.md)
* [Actividad de Pig](transform-data-using-hadoop-pig.md)
* [Actividad de MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Actividad de streaming de Hadoop](transform-data-using-hadoop-streaming.md)
* [Actividad de Spark](transform-data-using-spark.md)
* [Actividad de ejecución por lotes de Azure Machine Learning Studio (clásico)](transform-data-using-machine-learning.md)
* [Actividad de procedimiento almacenado](transform-data-using-stored-procedure.md)
