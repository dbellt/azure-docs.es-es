---
title: " Solución de problemas con las consultas de diagnóstico"
titleSuffix: Azure Cosmos DB
description: Obtenga información sobre cómo consultar registros de diagnóstico para solucionar problemas de los datos almacenados en Azure Cosmos DB
author: StefArroyo
services: cosmos-db
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/12/2021
ms.author: esarroyo
ms.openlocfilehash: 51ab68d77e6d5f7e69701b7bc36eaf58f51bf48d
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111966356"
---
# <a name="troubleshoot-issues-with-diagnostics-queries"></a> Solución de problemas con las consultas de diagnóstico

En este artículo, explicaremos cómo escribir consultas sencillas para ayudar a solucionar problemas relacionados con su cuenta de Azure Cosmos DB mediante registros de diagnóstico enviados a tablas de **AzureDiagnostics (heredadas)** y **específicas del recurso (versión preliminar)** .

En el caso de las tablas de Azure Diagnostics, todos los datos se escriben en una sola tabla y los usuarios tendrán que especificar la categoría que desean consultar.

En el caso de las tablas específicas del recurso (actualmente en versión preliminar para SQL API), los datos se escriben en tablas individuales para cada categoría del recurso. Recomendamos este modo, ya que facilita el trabajo con los datos, proporciona una mejor detectabilidad de los esquemas y mejora el rendimiento tanto en la latencia de ingesta como en los tiempos de consulta.

## <a name="azurediagnostics-queries"></a><a id="azure-diagnostics-queries"></a> Consultas AzureDiagnostics

- Cómo puedo consultar las operaciones que tardan más de 3 milisegundos en ejecutarse:

   ```Kusto
   AzureDiagnostics 
   | where toint(duration_s) > 3 and ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" 
   | summarize count() by clientIpAddress_s, TimeGenerated
   ```

- Cómo puedo consultar el agente de usuario que ejecuta las operaciones:

   ```Kusto
   AzureDiagnostics 
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" 
   | summarize count() by OperationName, userAgent_s
   ```

- Cómo puedo consultar las operaciones de larga duración:

   ```Kusto
   AzureDiagnostics 
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" 
   | project TimeGenerated , duration_s 
   | summarize count() by bin(TimeGenerated, 5s)
   | render timechart
   ```
    
- Cómo puedo obtener estadísticas de clave de partición para evaluar el sesgo entre las tres primeras particiones de una cuenta de base de datos:

   ```Kusto
   AzureDiagnostics 
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="PartitionKeyStatistics" 
   | project SubscriptionId, regionName_s, databaseName_s, collectionName_s, partitionKey_s, sizeKb_d, ResourceId 
   ```

- ¿Cómo puedo obtener los cargos de solicitud para las consultas costosas?

   ```Kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" and todouble(requestCharge_s) > 10.0
   | project activityId_g, requestCharge_s
   | join kind= inner (
   AzureDiagnostics
   | where ResourceProvider =="MICROSOFT.DOCUMENTDB" and Category == "QueryRuntimeStatistics"
   | project activityId_g, querytext_s
   ) on $left.activityId_g == $right.activityId_g
   | order by requestCharge_s desc
   | limit 100
   ```

- ¿Cómo puedo averiguar qué operaciones consumen la mayor parte de las RU por segundo?

    ```Kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests"
   | where TimeGenerated >= ago(2h) 
   | summarize max(responseLength_s), max(requestLength_s), max(requestCharge_s), count = count() by OperationName, requestResourceType_s, userAgent_s, collectionRid_s, bin(TimeGenerated, 1h)
   ```

- Cómo obtener todas las consultas que utilizan más de 100 RU/s en combinación con datos de **DataPlaneRequests** y **QueryRunTimeStatistics**.

   ```kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests" and todouble(requestCharge_s) > 100.0
   | project activityId_g, requestCharge_s
   | join kind= inner (
           AzureDiagnostics
           | where ResourceProvider =="MICROSOFT.DOCUMENTDB" and Category == "QueryRuntimeStatistics"
           | project activityId_g, querytext_s
   ) on $left.activityId_g == $right.activityId_g
   | order by requestCharge_s desc
   | limit 100
   ```

- ¿Cómo puedo obtener los cargos de una solicitud y la duración de la ejecución de una consulta?

   ```kusto
   AzureDiagnostics
   | where TimeGenerated >= ago(24hr)
   | where Category == "QueryRuntimeStatistics"
   | join (
   AzureDiagnostics
   | where TimeGenerated >= ago(24hr)
   | where Category == "DataPlaneRequests"
   ) on $left.activityId_g == $right.activityId_g
   | project databasename_s, collectionname_s, OperationName1 , querytext_s,requestCharge_s1, duration_s1, bin(TimeGenerated, 1min)
   ```

- ¿Cómo puedo obtener la distribución de distintas operaciones?

   ```kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests"
   | where TimeGenerated >= ago(2h) 
   | summarize count = count()  by OperationName, requestResourceType_s, bin(TimeGenerated, 1h) 
   ```

- ¿Cuál es el rendimiento máximo que ha consumido una partición?
   
   ```kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests"
   | where TimeGenerated >= ago(2h) 
   | summarize max(requestCharge_s) by bin(TimeGenerated, 1h), partitionId_g
   ```

- ¿Cómo puedo obtener la información sobre el consumo de RU por segundo de las claves de partición?

   ```kusto
   AzureDiagnostics 
   | where ResourceProvider == "MICROSOFT.DOCUMENTDB" and Category == "PartitionKeyRUConsumption" 
   | summarize total = sum(todouble(requestCharge_s)) by databaseName_s, collectionName_s, partitionKey_s, TimeGenerated 
   | order by TimeGenerated asc 
   ```

- Cómo obtener el cargo de la solicitud de una clave de partición específica

   ```kusto
   AzureDiagnostics 
   | where ResourceProvider == "MICROSOFT.DOCUMENTDB" and Category == "PartitionKeyRUConsumption" 
   | where parse_json(partitionKey_s)[0] == "2" 
   ```

- ¿Cómo puedo obtener las principales claves de partición con el mayor número de RU por segundo consumidas en un período específico?

   ```kusto
   AzureDiagnostics 
   | where ResourceProvider == "MICROSOFT.DOCUMENTDB" and Category == "PartitionKeyRUConsumption" 
   | where TimeGenerated >= datetime("11/26/2019, 11:20:00.000 PM") and TimeGenerated <= datetime("11/26/2019, 11:30:00.000 PM") 
   | summarize total = sum(todouble(requestCharge_s)) by databaseName_s, collectionName_s, partitionKey_s 
   | order by total desc
   ```

- ¿Cómo puedo obtener los registros de las claves de partición cuyo tamaño de almacenamiento es superior a los 8 GB?

   ```kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="PartitionKeyStatistics"
   | where todouble(sizeKb_d) > 800000
   ```

- ¿Cómo puedo obtener latencias de replicación P99 o P50 en las operaciones, los cargos de solicitud o la longitud de la respuesta?

   ```kusto
   AzureDiagnostics
   | where ResourceProvider=="MICROSOFT.DOCUMENTDB" and Category=="DataPlaneRequests"
   | where TimeGenerated >= ago(2d)
   | summarize percentile(todouble(responseLength_s), 50), percentile(todouble(responseLength_s), 99), max(responseLength_s), percentile(todouble(requestCharge_s), 50), percentile(todouble(requestCharge_s), 99), max(requestCharge_s), percentile(todouble(duration_s), 50), percentile(todouble(duration_s), 99), max(duration_s), count() by OperationName, requestResourceType_s, userAgent_s, collectionRid_s, bin(TimeGenerated, 1h)
   ```
 
- ¿Cómo se obtienen los registros de ControlPlane?
 
   Recuerde activar la marca tal como se describe en el artículo [Deshabilitar el acceso de escritura de metadatos basados en claves](audit-control-plane-logs.md#disable-key-based-metadata-write-access) y ejecutar las operaciones con Azure PowerShell, la CLI de Azure o Azure Resource Manager.
 
   ```kusto  
   AzureDiagnostics 
   | where Category =="ControlPlaneRequests"
   | summarize by OperationName 
   ```


## <a name="resource-specific-queries"></a><a id="resource-specific-queries"></a> Consultas específicas del recurso

- Cómo puedo consultar las operaciones que tardan más de 3 milisegundos en ejecutarse:

    ```kusto
    CDBDataPlaneRequests 
    | where toint(DurationMs) > 3
    | summarize count() by ClientIpAddress, TimeGenerated
    ```

- Cómo puedo consultar el agente de usuario que ejecuta las operaciones:

   ```kusto
    CDBDataPlaneRequests
    | summarize count() by OperationName, UserAgent
   ```

- Cómo puedo consultar las operaciones de larga duración:

   ```kusto
    CDBDataPlaneRequests
    | project TimeGenerated , DurationMs 
    | summarize count() by bin(TimeGenerated, 5s)
    | render timechart
   ```
    
- Cómo puedo obtener estadísticas de clave de partición para evaluar el sesgo entre las tres primeras particiones de una cuenta de base de datos:

   ```kusto
    CDBPartitionKeyStatistics
    | project RegionName, DatabaseName, CollectionName, PartitionKey, SizeKb
   ```

- ¿Cómo puedo obtener los cargos de solicitud para las consultas costosas?

   ```kusto
    CDBDataPlaneRequests
    | where todouble(RequestCharge) > 10.0
    | project ActivityId, RequestCharge
    | join kind= inner (
    CDBQueryRuntimeStatistics
    | project ActivityId, QueryText
    ) on $left.ActivityId == $right.ActivityId
    | order by RequestCharge desc
    | limit 100
   ```

- ¿Cómo puedo averiguar qué operaciones consumen la mayor parte de las RU por segundo?

    ```kusto
    CDBDataPlaneRequests
    | where TimeGenerated >= ago(2h) 
    | summarize max(ResponseLength), max(RequestLength), max(RequestCharge), count = count() by OperationName, RequestResourceType, UserAgent, CollectionName, bin(TimeGenerated, 1h)
   ```

- Cómo obtener todas las consultas que utilizan más de 100 RU/s en combinación con datos de **DataPlaneRequests** y **QueryRunTimeStatistics**.

   ```kusto
    CDBDataPlaneRequests
    | where todouble(RequestCharge) > 100.0
    | project ActivityId, RequestCharge
    | join kind= inner (
    CDBQueryRuntimeStatistics
    | project ActivityId, QueryText
    ) on $left.ActivityId == $right.ActivityId
    | order by RequestCharge desc
    | limit 100
   ```

- ¿Cómo puedo obtener los cargos de una solicitud y la duración de la ejecución de una consulta?

   ```kusto
    CDBQueryRuntimeStatistics
    | join kind= inner (
    CDBDataPlaneRequests
    ) on $left.ActivityId == $right.ActivityId
    | project DatabaseName, CollectionName, OperationName , QueryText, RequestCharge, DurationMs, bin(TimeGenerated, 1min)
   ```

- ¿Cómo puedo obtener la distribución de distintas operaciones?

   ```kusto
    CDBDataPlaneRequests
    | where TimeGenerated >= ago(2h) 
    | summarize count = count()  by OperationName, RequestResourceType, bin(TimeGenerated, 1h)
   ```

- ¿Cuál es el rendimiento máximo que ha consumido una partición?

   ```kusto
   CDBDataPlaneRequests
   | where TimeGenerated >= ago(2h) 
   | summarize max(RequestCharge) by bin(TimeGenerated, 1h), PartitionId
   ```

- ¿Cómo puedo obtener la información sobre el consumo de RU por segundo de las claves de partición?

   ```kusto
   CDBPartitionKeyRUConsumption 
   | summarize total = sum(todouble(RequestCharge)) by DatabaseName, CollectionName, PartitionKey, TimeGenerated 
   | order by TimeGenerated asc 
   ```

- ¿Cómo puedo obtener el cargo de la solicitud de una clave de partición específica?

   ```kusto
   CDBPartitionKeyRUConsumption  
   | where parse_json(PartitionKey)[0] == "2" 
   ```

- ¿Cómo puedo obtener las principales claves de partición con el mayor número de RU por segundo consumidas en un período específico? 

   ```kusto
   CDBPartitionKeyRUConsumption
   | where TimeGenerated >= datetime("02/12/2021, 11:20:00.000 PM") and TimeGenerated <= datetime("05/12/2021, 11:30:00.000 PM") 
   | summarize total = sum(todouble(RequestCharge)) by DatabaseName, CollectionName, PartitionKey 
   | order by total desc
   ```

- ¿Cómo puedo obtener los registros de las claves de partición cuyo tamaño de almacenamiento es superior a los 8 GB?

   ```kusto
   CDBPartitionKeyStatistics
   | where todouble(SizeKb) > 800000
   ``` 

- ¿Cómo puedo obtener latencias de replicación P99 o P50 en las operaciones, los cargos de solicitud o la longitud de la respuesta?

   ```kusto
   CDBDataPlaneRequests
   | where TimeGenerated >= ago(2d)
   | summarize percentile(todouble(ResponseLength), 50), percentile(todouble(ResponseLength), 99), max(ResponseLength), percentile(todouble(RequestCharge), 50), percentile(todouble(RequestCharge), 99), max(RequestCharge), percentile(todouble(DurationMs), 50), percentile(todouble(DurationMs), 99), max(DurationMs),count() by OperationName, RequestResourceType, UserAgent, CollectionName, bin(TimeGenerated, 1h)
   ```
 
- ¿Cómo se obtienen los registros de ControlPlane?
 
   Recuerde activar la marca tal como se describe en el artículo [Deshabilitar el acceso de escritura de metadatos basados en claves](audit-control-plane-logs.md#disable-key-based-metadata-write-access) y ejecutar las operaciones con Azure PowerShell, la CLI de Azure o Azure Resource Manager.
 
   ```kusto  
   CDBControlPlaneRequests
   | summarize by OperationName 
   ```

## <a name="next-steps"></a>Pasos siguientes
* Para obtener más información sobre cómo crear una configuración de diagnóstico para Cosmos DB, consulte el artículo sobre cómo [crear una configuración de diagnóstico](cosmosdb-monitor-resource-logs.md).

* Para obtener información detallada sobre cómo crear una configuración de diagnóstico mediante Azure Portal, la CLI o PowerShell, consulte el artículo [Creación de una configuración de diagnóstico para recopilar registros de plataforma y métricas en Azure](../azure-monitor/essentials/diagnostic-settings.md).