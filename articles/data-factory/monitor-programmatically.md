---
title: Supervisión mediante programación de una factoría de datos de Azure
description: Aprenda a supervisar una canalización en una factoría de datos mediante distintos kits de desarrollo de software (SDK).
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/16/2018
author: minhe-msft
ms.author: hemin
ms.custom: devx-track-python
ms.openlocfilehash: 4c6c54138f7966c505ec0cdea2a90d2a8c1655f8
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112005526"
---
# <a name="programmatically-monitor-an-azure-data-factory"></a>Supervisión mediante programación de una factoría de datos de Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se describe cómo supervisar una canalización de una factoría de datos mediante distintos kits de desarrollo de software (SDK). 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="data-range"></a>Intervalo de datos

Data Factory solo almacena los datos de ejecución de canalización durante 45 días. Al consultar mediante programación los datos sobre las ejecuciones de canalización de Data Factory (por ejemplo, con el comando de PowerShell `Get-AzDataFactoryV2PipelineRun`) no hay ninguna fecha máxima para los parámetros opcionales `LastUpdatedAfter` y `LastUpdatedBefore`. No obstante, si consulta los datos del año pasado, por ejemplo, no obtendrá un error, sino solo los datos de ejecución de canalización de los últimos 45 días.

Si desea conservar los datos de ejecución de canalización durante más de 45 días, configure su propio registro de diagnóstico con [Azure Monitor](monitor-using-azure-monitor.md).

## <a name="pipeline-run-information"></a>Información de la ejecución de canalización

Para conocer las propiedades de la ejecución de canalización, consulte la [referencia de la API de PipelineRun](/rest/api/datafactory/pipelineruns/get#pipelinerun). Una ejecución de canalización tiene un estado diferente durante su ciclo de vida. Los valores posibles del estado de ejecución se enumeran a continuación:

* En cola
* InProgress
* Correcto
* Con error
* Cancelando
* Canceled

## <a name="net"></a>.NET
Para ver un tutorial completo sobre cómo crear y supervisar una canalización mediante el SDK de .NET, consulte [Creación de una factoría de datos y una canalización con SDK de .NET](quickstart-create-data-factory-dot-net.md).

1. Agregue el código siguiente para comprobar continuamente el estado de la ejecución de canalización hasta que termine de copiar los datos.

    ```csharp
    // Monitor the pipeline run
    Console.WriteLine("Checking pipeline run status...");
    PipelineRun pipelineRun;
    while (true)
    {
        pipelineRun = client.PipelineRuns.Get(resourceGroup, dataFactoryName, runResponse.RunId);
        Console.WriteLine("Status: " + pipelineRun.Status);
        if (pipelineRun.Status == "InProgress" || pipelineRun.Status == "Queued")
            System.Threading.Thread.Sleep(15000);
        else
            break;
    }
    ```

2. Agregue el código siguiente que recupera detalles de la ejecución de actividad de copia, como el tamaño de los datos leídos o escritos.

    ```csharp
    // Check the copy activity run details
    Console.WriteLine("Checking copy activity run details...");

    RunFilterParameters filterParams = new RunFilterParameters(
        DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10));
    ActivityRunsQueryResponse queryResponse = client.ActivityRuns.QueryByPipelineRun(
        resourceGroup, dataFactoryName, runResponse.RunId, filterParams);
    if (pipelineRun.Status == "Succeeded")
        Console.WriteLine(queryResponse.Value.First().Output);
    else
        Console.WriteLine(queryResponse.Value.First().Error);
    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

Para obtener la documentación completa del SDK de .NET. consulte la [referencia del SDK de .NET de Data Factory](/dotnet/api/microsoft.azure.management.datafactory).

## <a name="python"></a>Python
Para ver un tutorial completo sobre cómo crear y supervisar una canalización mediante el SDK de Python, consulte [Creación de una factoría de datos y una canalización con Python](quickstart-create-data-factory-python.md).

Para supervisar la ejecución de canalización, agregue el código siguiente:

```python
# Monitor the pipeline run
time.sleep(30)
pipeline_run = adf_client.pipeline_runs.get(
    rg_name, df_name, run_response.run_id)
print("\n\tPipeline run status: {}".format(pipeline_run.status))
filter_params = RunFilterParameters(
    last_updated_after=datetime.now() - timedelta(1), last_updated_before=datetime.now() + timedelta(1))
query_response = adf_client.activity_runs.query_by_pipeline_run(
    rg_name, df_name, pipeline_run.run_id, filter_params)
print_activity_run_details(query_response.value[0])
```

Para obtener la documentación completa del SDK de .NET. consulte la [referencia del SDK de Python de Data Factory](/python/api/overview/azure/datafactory).

## <a name="rest-api"></a>API DE REST
Para ver un tutorial completo sobre cómo crear y supervisar una canalización mediante la API de REST, consulte [Creación de una instancia de Azure Data Factory y una canalización mediante la API de REST](quickstart-create-data-factory-rest-api.md).
 
1. Ejecute el script siguiente para comprobar continuamente el estado de ejecución de la canalización hasta que termine de copiar los datos.

    ```powershell
    $request = "https://management.azure.com/subscriptions/${subsId}/resourceGroups/${resourceGroup}/providers/Microsoft.DataFactory/factories/${dataFactoryName}/pipelineruns/${runId}?api-version=${apiVersion}"
    while ($True) {
        $response = Invoke-RestMethod -Method GET -Uri $request -Header $authHeader
        Write-Host  "Pipeline run status: " $response.Status -foregroundcolor "Yellow"

        if ( ($response.Status -eq "InProgress") -or ($response.Status -eq "Queued") ) {
            Start-Sleep -Seconds 15
        }
        else {
            $response | ConvertTo-Json
            break
        }
    }
    ```
2. Ejecute el script siguiente para recuperar detalles de la ejecución de la actividad de copia, como el tamaño de los datos leídos o escritos.

    ```powershell
    $request = "https://management.azure.com/subscriptions/${subscriptionId}/resourceGroups/${resourceGroupName}/providers/Microsoft.DataFactory/factories/${factoryName}/pipelineruns/${runId}/queryActivityruns?api-version=${apiVersion}&startTime="+(Get-Date).ToString('yyyy-MM-dd')+"&endTime="+(Get-Date).AddDays(1).ToString('yyyy-MM-dd')+"&pipelineName=Adfv2QuickStartPipeline"
    $response = Invoke-RestMethod -Method POST -Uri $request -Header $authHeader
    $response | ConvertTo-Json
    ```

Para obtener la documentación completa sobre la API de REST, consulte la [referencia de la API de REST de Data Factory](/rest/api/datafactory/).

## <a name="powershell"></a>PowerShell
Para ver un tutorial completo sobre cómo crear y supervisar una canalización mediante PowerShell, consulte [Creación de una instancia de Azure Data Factory con PowerShell](quickstart-create-data-factory-powershell.md).

1. Ejecute el script siguiente para comprobar continuamente el estado de ejecución de la canalización hasta que termine de copiar los datos.

    ```powershell
    while ($True) {
        $run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $resourceGroupName -DataFactoryName $DataFactoryName -PipelineRunId $runId

        if ($run) {
            if ( ($run.Status -ne "InProgress") -and ($run.Status -ne "Queued") ) {
                Write-Output ("Pipeline run finished. The status is: " +  $run.Status)
                $run
                break
            }
            Write-Output ("Pipeline is running...status: " + $run.Status)
        }

        Start-Sleep -Seconds 30
    }
    ```
2. Ejecute el script siguiente para recuperar detalles de la ejecución de la actividad de copia, como el tamaño de los datos leídos o escritos.

    ```powershell
    Write-Host "Activity run details:" -foregroundcolor "Yellow"
    $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    $result
    
    Write-Host "Activity 'Output' section:" -foregroundcolor "Yellow"
    $result.Output -join "`r`n"
    
    Write-Host "\nActivity 'Error' section:" -foregroundcolor "Yellow"
    $result.Error -join "`r`n"
    ```

Para obtener la documentación completa sobre los cmdlets de PowerShell, consulte la [referencia de los cmdlets de PowerShell de Data Factory](/powershell/module/az.datafactory).

## <a name="next-steps"></a>Pasos siguientes
Consulte el artículo sobre la [supervisión de canalizaciones mediante Azure Monitor](monitor-using-azure-monitor.md) para obtener información sobre cómo usar Azure Monitor para supervisar las canalizaciones de factoría de datos.