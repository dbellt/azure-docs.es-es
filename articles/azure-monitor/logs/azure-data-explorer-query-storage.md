---
title: Consulta de datos exportados desde Azure Monitor mediante Azure Data Explorer (versión preliminar)
description: Use Azure Data Explorer para consultar datos exportados del área de trabajo de Log Analytics a una cuenta de Azure Storage.
author: osalzberg
ms.author: bwren
ms.reviewer: bwren
ms.topic: conceptual
ms.date: 10/13/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ad938d15f8e21ed34014c0a743b5ba891f5476e0
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108316846"
---
# <a name="query-exported-data-from-azure-monitor-using-azure-data-explorer-preview"></a>Consulta de datos exportados desde Azure Monitor mediante Azure Data Explorer (versión preliminar)
Exportar datos de Azure Monitor a una cuenta de Azure Storage permite la retención a bajo costo y la capacidad de reasignar registros a regiones distintas. Use Azure Data Explorer para consultar datos exportados de las áreas de trabajo de Log Analytics. Una vez que se configuran, las tablas compatibles que se envían desde las áreas de trabajo hasta la cuenta de Azure Storage estarán disponibles como un origen de datos para Azure Data Explorer.

El flujo del proceso es el siguiente: 

1.  Exportar datos desde un área de trabajo de Log Analytics hasta una cuenta de Azure Storage.
2.  Crear una tabla externa en el clúster de Azure Data Explorer y una asignación para los tipos de datos.
3.  Consultar datos desde Azure Data Explorer.

:::image type="content" source="media/azure-data-explorer-query-storage/exported-data-query.png" alt-text="Flujo de consultas de datos exportados de Azure Data Explorer.":::



## <a name="send-data-to-azure-storage"></a>Envío de datos a Azure Storage
Los registros de Azure Monitor se pueden exportar a una cuenta de Azure Storage mediante cualquiera de estas opciones.

- Para exportar todos los datos del área de trabajo de Log Analytics a una cuenta de Azure Storage o un centro de eventos, use la característica de exportación de datos del área de trabajo de Log Analytics de Azure Monitor Logs. Consulte [Exportación de datos del área de trabajo de Log Analytics en Azure Monitor (versión preliminar)](./logs-data-export.md).
- Exportación programada desde una consulta de registro con una aplicación lógica. Es similar a la característica de exportación de datos, pero permite enviar datos filtrados o agregados a Azure Storage. Aunque este método está sujeto a [límites de consultas de registro](../service-limits.md#log-analytics-workspaces). Consulte [Archivado de datos de un área de trabajo de Log Analytics a Azure Storage mediante Logic Apps](./logs-export-logic-app.md).
- Exportación única mediante una aplicación lógica. Consulte [Conector de Azure Monitor Logs para Logic Apps y Power Automate](./logicapp-flow-connector.md).
- Exportación única a la máquina local mediante el script de PowerShell. Consulte [Invoke-AzOperationalInsightsQueryExport](https://www.powershellgallery.com/packages/Invoke-AzOperationalInsightsQueryExport).

> [!TIP]
> Puede usar un clúster de Azure Data Explorer o crear un clúster dedicado nuevo con las configuraciones necesarias.

## <a name="create-an-external-table-located-in-azure-blob-storage"></a>Creación de una tabla externa ubicada en Azure Blob Storage
Use [tablas externas](/azure/data-explorer/kusto/query/schema-entities/externaltables) para vincular Azure Data Explorer a una cuenta de Azure Storage. Una tabla externa es una entidad de esquema de Kusto que hace referencia a los datos almacenados fuera de una base de datos de Kusto. Al igual que las tablas, una tabla externa tiene un esquema bien definido. A diferencia de las tablas, los datos se almacenan y administran fuera de un clúster de Kusto. Los datos exportados de la sección anterior se guardan en líneas JSON.

Para crear una referencia, necesita el esquema de la tabla exportada. Use el operador [getschema](/azure/data-explorer/kusto/query/getschemaoperator) de Log Analytics para recuperar esta información que incluye las columnas de la tabla y sus tipos de datos.

:::image type="content" source="media\azure-data-explorer-query-storage\exported-data-map-schema.jpg" alt-text="Esquema de tablas de Log Analytics.":::

Ahora puede usar la salida para crear la consulta de Kusto para compilar la tabla externa.
Siga las instrucciones que aparecen en [Creación y modificación de tablas externas en Azure Storage o Azure Data Lake](/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake), cree una tabla externa en formato JSON y, luego, ejecute la consulta desde la base de datos de Azure Data Explorer.

>[!NOTE]
>La creación de la tabla externa se genera a partir de dos procesos. El primero es la creación de la tabla externa, mientras que el segundo es la creación de la asignación.

El script de PowerShell siguiente creará los comandos [create](/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake#create-external-table-mapping) para la tabla y la asignación.

```powershell
PARAM(
    $resourcegroupname, #The name of the Azure resource group
    $TableName, # The Log Analytics table you wish to convert to external table
    $MapName, # The name of the map
    $subscriptionId, #The ID of the subscription
    $WorkspaceId, # The Log Analytics WorkspaceId
    $WorkspaceName, # The Log Analytics workspace name
    $BlobURL, # The Blob URL where the data is saved
    $ContainerAccessKey, # The blob container Access Key (Option to add a SAS url)
    $ExternalTableName = $null # The External Table name, null to use the same name
)

if($null -eq $ExternalTableName) {
    $ExternalTableName = $TableName
}

$query = $TableName + ' | getschema | project ColumnName, DataType'

$output = (Invoke-AzOperationalInsightsQuery -WorkspaceId $WorkspaceId -Query $query).Results

$FirstCommand = @()
$SecondCommand = @()

foreach ($record in $output) {
    if ($record.DataType -eq 'System.DateTime') {
        $dataType = 'datetime'
    } else {
        $dataType = 'string'
    }
    $FirstCommand += $record.ColumnName + ":" + "$dataType" + ","
    $SecondCommand += "{`"column`":" + "`"" + $record.ColumnName + "`"," + "`"datatype`":`"$dataType`",`"path`":`"$." + $record.ColumnName + "`"},"
}
$schema = ($FirstCommand -join '') -replace ',$'
$mapping = ($SecondCommand -join '') -replace ',$'

$CreateExternal = @'
.create external table {0} ({1})
kind=blob
partition by (TimeGeneratedPartition:datetime = bin(TimeGenerated, 1min))
pathformat = (datetime_pattern("'y='yyyy'/m='MM'/d='dd'/h='HH'/m='mm", TimeGeneratedPartition))
dataformat=multijson
(
   h@'{2}/WorkspaceResourceId=/subscriptions/{4}/resourcegroups/{6}/providers/microsoft.operationalinsights/workspaces/{5};{3}'
)
with
(
   docstring = "Docs",
   folder = "ExternalTables"
)
'@ -f $TableName, $schema, $BlobURL, $ContainerAccessKey, $subscriptionId, $WorkspaceName, $resourcegroupname,$WorkspaceId

$createMapping = @'
.create external table {0} json mapping "{1}" '[{2}]'
'@ -f $ExternalTableName, $MapName, $mapping

Write-Host -ForegroundColor Red 'Copy and run the following commands (one by one), on your Azure Data Explorer cluster query window to create the external table and mappings:'
write-host -ForegroundColor Green $CreateExternal
Write-Host -ForegroundColor Green $createMapping
```

En la imagen siguiente se muestra un ejemplo de la salida.

:::image type="content" source="media/azure-data-explorer-query-storage/external-table-create-command-output.png" alt-text="Salida del comando ExternalTable create.":::

[![Salida de ejemplo](media/azure-data-explorer-query-storage/external-table-create-command-output.png)](media/azure-data-explorer-query-storage/external-table-create-command-output.png#lightbox)

>[!TIP]
>* Copie, pegue y ejecute la salida del script en la herramienta de cliente de Azure Data Explorer para crear la tabla y la asignación.
>* Para utilizar todos los datos dentro del contenedor, modifique el script y cambie la dirección URL para que sea 'https://your.blob.core.windows.net/containername;SecKey'

## <a name="query-the-exported-data-from-azure-data-explorer"></a>Consulta de los datos exportados desde Azure Data Explorer 

Después de configurar la asignación, puede consultar los datos exportados desde Azure Data Explorer. La consulta requiere la función [external_table](/azure/data-explorer/kusto/query/externaltablefunction) como se muestra en el ejemplo siguiente.

```kusto
external_table("HBTest","map") | take 10000
```

[![Consulta de datos exportados de Log Analytics](media/azure-data-explorer-query-storage/external-table-query.png)](media/azure-data-explorer-query-storage/external-table-query.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [escribir consultas en Azure Data Explorer](/azure/data-explorer/write-queries)
