---
title: Solución de problemas de las excepciones de tasa de solicitudes demasiado grande en Azure Cosmos DB
description: Aprenda a diagnosticar y corregir las excepciones de tasa de solicitudes demasiado grande.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: ed75ad96346d4a98e947a6231714d19431aa2715
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109734401"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-request-rate-too-large-429-exceptions"></a>Diagnóstico y solución de problemas de las excepciones de tasa de solicitudes demasiado grande (429) en Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Este artículo contiene las causas conocidas y las soluciones de varios errores de código de estado 429 para la API de SQL. Si usa la API para MongoDB, consulte el artículo [Solución de problemas comunes en API para MongoDB](mongodb-troubleshoot.md) para obtener información sobre cómo depurar el código de estado 16500.

Una excepción de "tasa de solicitudes demasiado grande", también conocida como código de error 429, indica que se limita la tasa de las solicitudes en Azure Cosmos DB.

Cuando se usa el rendimiento aprovisionado, se establece el rendimiento medido en unidades de solicitud por segundo (RU/s) necesario para la carga de trabajo. Las operaciones de base de datos en el servicio, como lecturas, escrituras y consultas, consumen cierta cantidad de unidades de solicitud (RU). Más información sobre las [unidades de solicitud](request-units.md).

En un segundo determinado, si las operaciones consumen más que las unidades de solicitud aprovisionadas, Azure Cosmos DB devolverá una excepción 429. Cada segundo, se restablece el número de unidades de solicitud disponibles para su uso.

Antes de realizar una acción para cambiar las RU/s, es importante comprender la causa principal de la limitación de tasa y solucionar el problema subyacente.  

Hay diferentes mensajes de error que corresponden a diferentes tipos de excepciones 429:
- [La tasa de solicitudes es grande. Es posible que se necesiten más unidades de solicitud, por lo que no se ha realizado ningún cambio.](#request-rate-is-large)
- [La solicitud no se ha completado debido a una alta tasa de solicitudes de metadatos.](#rate-limiting-on-metadata-requests)
- [La solicitud no se ha completado debido a un error transitorio del servicio.](#rate-limiting-due-to-transient-service-error)


## <a name="request-rate-is-large"></a>La tasa de solicitudes es grande
Se trata del escenario más habitual. Se produce cuando las unidades de solicitud consumidas por las operaciones en los datos superan el número aprovisionado de RU/s. 

### <a name="step-1-check-the-metrics-to-determine-the-percentage-of-requests-with-429-error"></a>Paso 1: Comprobación de las métricas para determinar el porcentaje de solicitudes con el error 429
Consultar los mensajes del error 429 no significa necesariamente que haya un problema con la base de datos o el contenedor.

#### <a name="how-to-investigate"></a>Investigación

Determine qué porcentaje de las solicitudes para la base de datos o el contenedor ha resultado en errores 429, en comparación con el recuento total de solicitudes correctas. En la hoja de la cuenta de Azure Cosmos DB, vaya a **Insights** > **Solicitudes** > **Total Requests by Status Code** (Total de solicitudes por código de estado). Filtre por una base de datos y un contenedor específicos. 

De manera predeterminada, los SDK de cliente de Azure Cosmos DB y las herramientas de importación de datos, como Azure Data Factory y la biblioteca de Bulk Executor, reintentan automáticamente las solicitudes con errores 429. Normalmente, las reintenten hasta 9 veces. Como resultado, aunque pueda ver errores 429 en las métricas, es posible que estos errores ni siquiera se hayan devuelto a la aplicación. 

:::image type="content" source="media/troubleshoot-request-rate-too-large/insights-429-requests.png" alt-text="Total de solicitudes por código de estado que muestra el número de solicitudes con errores 429 y 2xx.":::


#### <a name="recommended-solution"></a>Solución recomendada
En general, para una carga de trabajo de producción, si ve entre el 1 y el 5 % de las solicitudes con errores 429 y la latencia de un extremo a otro es aceptable, se trata de una señal correcta de que las RU/s se están empleando completamente. No se requiere ninguna acción. De lo contrario, vaya a los siguientes pasos de solución de problemas.

### <a name="step-2-determine-if-there-is-a-hot-partition"></a>Paso 2: Determinación de si hay una partición de nivel de acceso frecuente
Una partición de nivel de acceso frecuente surge cuando una o varias claves de partición lógica consumen una cantidad desproporcionada del total de RU/s debido a un mayor volumen de solicitudes. Esto puede deberse a un diseño de clave de partición que no distribuye uniformemente las solicitudes. Como resultado, muchas solicitudes se dirigen a un pequeño subconjunto de particiones lógicas (lo que implica a las físicas) que pasan a ser de "nivel de acceso frecuente". Dado que todos los datos de una partición lógica residen en una partición física y el total de RU/s se distribuye uniformemente entre las particiones físicas, una partición de nivel de acceso frecuente puede dar lugar a errores 429 y a un uso ineficaz del rendimiento. 

Estos son algunos ejemplos de estrategias de creación de particiones que conducen a particiones de nivel de acceso frecuente:
- Tiene un contenedor que almacena datos de dispositivo IoT para una carga de trabajo con mucha escritura que se particiona por fecha. Todos los datos de una sola fecha residirán en la misma partición lógica y física. Dado que todos los datos escritos cada día tienen la misma fecha, dará lugar a una partición de nivel de acceso frecuente todos los días. 
    - En su lugar, para este escenario, una clave de partición como un identificador (ya sea un GUID o un id. de dispositivo) o una [clave de partición sintética](/synthetic-partition-keys.md) que combine el identificador y la fecha produciría una mayor cardinalidad de valores y una mejor distribución del volumen de solicitudes.
- Tiene un escenario multiinquilino con un contenedor particionado por tenantId. Si un inquilino está considerablemente más activo que los demás, se produce una partición de nivel de acceso frecuente. Por ejemplo, si el inquilino más grande tiene 100 000 usuarios, pero la mayoría de los inquilinos tienen menos de 10 usuarios, tendrá una partición de nivel de acceso frecuente cuando se particione por tenantID. 
    - En el escenario anterior, considere la posibilidad de tener un contenedor dedicado para el inquilino más grande, particionado por una propiedad más granular, como UserId. 
    
#### <a name="how-to-identify-the-hot-partition"></a>Identificación de la partición de nivel de acceso frecuente

Para comprobar si hay una partición de nivel de acceso frecuente, vaya a **Insights** > **Rendimiento** > **Normalized RU Consumption (%) By PartitionKeyRangeID** [Consumo de RU normalizado (%) por PartitionKeyRangeID]. Filtre por una base de datos y un contenedor específicos. 

Cada objeto PartitionKeyRangeId se asigna a una partición física. Si hay un objeto PartitionKeyRangeId cuyo consumo de RU normalizado es significativamente mayor que el de los demás (por ejemplo, es del 100 %, pero el de los demás es del 30 % o menos), puede ser un signo de que se trata de una partición de nivel de acceso frecuente. Obtenga más información sobre la [métrica de consumo de RU normalizado](monitor-normalized-request-units.md).

:::image type="content" source="media/troubleshoot-request-rate-too-large/split-norm-utilization-by-pkrange-hot-partition.png" alt-text="Gráfico del consumo de RU normalizado por PartitionKeyRangeId con una partición de nivel de acceso frecuente.":::

Para ver qué claves de partición lógica consumen más RU/s, use [Registros de diagnóstico de Azure](cosmosdb-monitor-resource-logs.md). Esta consulta de ejemplo suma el total de unidades de solicitud consumidas por segundo en cada clave de partición lógica. 

> [!IMPORTANT]
> La habilitación de los registros de diagnóstico conlleva un cargo independiente para el servicio Log Analytics, que se factura en función del volumen de datos ingeridos. Se recomienda activar los registros de diagnóstico durante un período limitado de tiempo para la depuración y desactivarlos cuando ya no sean necesarios. Vea la [página de precios](https://azure.microsoft.com/pricing/details/monitor/) para obtener más detalles.

```kusto
AzureDiagnostics
| where TimeGenerated >= ago(24hour)
| where Category == "PartitionKeyRUConsumption"
| where collectionName_s == "CollectionName" 
| where isnotempty(partitionKey_s)
// Sum total request units consumed by logical partition key for each second
| summarize sum(todouble(requestCharge_s)) by partitionKey_s, operationType_s, bin(TimeGenerated, 1s)
| order by sum_requestCharge_s desc
```
Esta salida de ejemplo muestra que, en un minuto determinado, la clave de partición lógica con el valor "Contoso" consumió aproximadamente 12 000 RU/s, mientras que la clave de partición lógica con el valor "Fabrikam" consumió menos de 600 RU/s. Si este patrón es continuo durante el período de tiempo en el que se produjo la limitación de tasa, indica una partición de nivel de acceso frecuente. 

:::image type="content" source="media/troubleshoot-request-rate-too-large/hot-logical-partition-key-results.png" alt-text="Claves de partición lógica que consumen la mayoría de las unidades de solicitud por segundo.":::

> [!TIP]
> En cualquier carga de trabajo, habrá una variación natural en el volumen de solicitudes entre particiones lógicas. Debe determinar si la partición de nivel de acceso frecuente se debe a un sesgo fundamental debido a la elección de la clave de partición (que puede requerir cambiar la clave) o a un pico temporal debido a una variación natural en los patrones de carga de trabajo.

#### <a name="recommended-solution"></a>Solución recomendada
Revise las instrucciones sobre [cómo elegir una buena clave de partición](/partitioning-overview.md#choose-partitionkey).

Si hay un alto porcentaje de solicitudes con limitación de tasa y no hay ninguna partición de nivel de acceso frecuente:
- Puede [aumentar las RU/s](set-throughput.md) en la base de datos o el contenedor mediante los SDK de cliente, Azure Portal, PowerShell, la CLI o la plantilla de ARM.  

Si hay un alto porcentaje de solicitudes con limitación de tasa y hay una partición de nivel de acceso frecuente subyacente:
-  A largo plazo, para obtener el mejor costo y rendimiento, considere la posibilidad de **cambiar la clave de partición**. La clave de partición no se puede actualizar localmente, por lo que es necesario migrar los datos a un nuevo contenedor con una clave de partición diferente. Azure Cosmos DB admite una [herramienta de migración de datos en directo](https://devblogs.microsoft.com/cosmosdb/how-to-change-your-partition-key/) para este propósito.
- A corto plazo, puede aumentar temporalmente las RU/s para permitir un mayor rendimiento de la partición de nivel de acceso frecuente. No se recomienda como una estrategia a largo plazo, ya que conduce a un sobreaprovisionado de RU/s y un costo mayor. 

> [!TIP]
>  Al aumentar el rendimiento, la operación de escalado vertical se completará de forma instantánea o requerirá entre 5 y 6 horas para completarse, en función del número de RU/s a las que quiera escalar verticalmente. Si quiere conocer el mayor número de RU/s que puede establecer sin desencadenar la operación asincrónica de escalado vertical (que requiere Azure Cosmos DB para aprovisionar más particiones físicas), multiplique el número de objetos PartitionKeyRangeId distintos por 10 000 RU/s. Por ejemplo, si tiene 30 000 RU/s aprovisionadas y 5 particiones físicas (6000 RU/s asignadas por partición física), puede aumentar a 50 000 RU/s (10 000 RU/s por partición física) en una operación de escalado vertical instantánea. Aumentar a >50 000 RU/s requerirá una operación asincrónica de escalado vertical.

### <a name="step-3-determine-what-requests-are-returning-429s"></a>Paso 3: Determinación de qué solicitudes devuelven errores 429

#### <a name="how-to-investigate-requests-with-429s"></a>Investigación de solicitudes con errores 429
Use [Registros de diagnóstico de Azure](cosmosdb-monitor-resource-logs.md) para identificar qué solicitudes devuelven errores 429 y cuántas RU consumieron. Esta consulta de ejemplo se agrega en el nivel de minuto. 

> [!IMPORTANT]
> La habilitación de los registros de diagnóstico conlleva un cargo independiente para el servicio Log Analytics, que se factura en función del volumen de datos ingeridos. Se recomienda activar los registros de diagnóstico durante un período limitado de tiempo para la depuración y desactivarlos cuando ya no sean necesarios. Vea la [página de precios](https://azure.microsoft.com/pricing/details/monitor/) para obtener más detalles.

```kusto
AzureDiagnostics
| where TimeGenerated >= ago(24h)
| where Category == "DataPlaneRequests"
| summarize throttledOperations = dcountif(activityId_g, statusCode_s == 429), totalOperations = dcount(activityId_g), totalConsumedRUPerMinute = sum(todouble(requestCharge_s)) by databaseName_s, collectionName_s, OperationName, requestResourceType_s, bin(TimeGenerated, 1min)
| extend averageRUPerOperation = 1.0 * totalConsumedRUPerMinute / totalOperations 
| extend fractionOf429s = 1.0 * throttledOperations / totalOperations
| order by fractionOf429s desc
```
Por ejemplo, esta salida de ejemplo muestra que en cada minuto, el 30 % de las solicitudes de creación de documentos tenían una limitación de tasa y que cada solicitud consumía un promedio de 17 RU.
:::image type="content" source="media/troubleshoot-request-rate-too-large/throttled-requests-diagnostic-logs-results.png" alt-text="Solicitudes con 429 en los registros de diagnóstico.":::

#### <a name="recommended-solution"></a>Solución recomendada
##### <a name="429s-on-create-replace-or-upsert-document-requests"></a>Errores 429 en las solicitudes de creación, reemplazo o upsert de documento
- De manera predeterminada, en la API de SQL, todas las propiedades se indexan de forma predeterminada. Ajuste la [directiva de indexación](index-policy.md) para indexar solo las propiedades necesarias.
Esto reducirá las unidades de solicitud necesarias por operación de creación de documento, lo que reducirá la probabilidad de ver errores 429 o le permitirá lograr más operaciones por segundo con la misma cantidad de RU/s aprovisionadas. 

##### <a name="429s-on-query-document-requests"></a>Errores 429 en solicitudes de consulta de documento
- Siga las instrucciones para [solucionar problemas de consultas con cargos elevados de RU](troubleshoot-query-performance.md#querys-ru-charge-is-too-high).

##### <a name="429s-on-execute-stored-procedures"></a>Errores 429 en la ejecución de procedimientos almacenados
- Los [procedimientos almacenados](stored-procedures-triggers-udfs.md) están previstos para las operaciones que requieren transacciones de escritura en un valor de clave de partición. No se recomienda usar procedimientos almacenados para un gran número de operaciones de lectura o consulta. Para obtener el mejor rendimiento, estas operaciones de lectura o consulta deben realizarse en el lado cliente, mediante los SDK de Cosmos. 

## <a name="rate-limiting-on-metadata-requests"></a>Limitación de tasa en las solicitudes de metadatos

La limitación de la tasa de metadatos puede producirse cuando se realizan un gran volumen de operaciones de metadatos en bases de datos o contenedores. Las operaciones de metadatos incluyen:
- Crear, leer, actualizar o eliminar un contenedor o una base de datos
- Enumerar las bases de datos o los contenedores de una cuenta de Cosmos
- Consultar las ofertas para ver el rendimiento aprovisionado actual 

Hay un límite de RU reservado por el sistema para estas operaciones, por lo que aumentar las RU/s aprovisionadas de la base de datos o el contenedor no tendrá ningún impacto y no se recomienda. Consulte los [límites de las operaciones de metadatos](concepts-limits.md#metadata-request-limits).

#### <a name="how-to-investigate"></a>Investigación
Vaya a **Insights** > **Sistema** > **Metadata Requests By Status Code** (Solicitudes de metadatos por código de estado). Filtre por una base de datos y un contenedor específicos, si lo desea. 

:::image type="content" source="media/troubleshoot-request-rate-too-large/metadata-throttling-insights.png" alt-text="Gráfico de solicitudes de metadatos por código de estado en Insights.":::

#### <a name="recommended-solution"></a>Solución recomendada
- Si la aplicación necesita realizar operaciones de metadatos, considere la posibilidad de implementar una directiva de retroceso para enviar estas solicitudes con una tasa menor. 

- Utilice instancias de cliente estáticas de Cosmos DB. Cuando se inicializa DocumentClient o CosmosClient, el SDK de Cosmos DB captura los metadatos sobre la cuenta, incluida la información sobre el nivel de coherencia, las bases de datos, los contenedores, las particiones y las ofertas. Esta inicialización puede consumir un gran número de RU y debe realizarse con poca frecuencia. Use una única instancia de DocumentClient y úsela durante toda la vigencia de la aplicación.

- Almacene en caché los nombres de las bases de datos y los contenedores. Durante el inicio, recupere los nombres de las bases de datos y los contenedores de la configuración o almacénelos en memoria caché. Llamadas como ReadDatabaseAsync/ReadDocumentCollectionAsync o CreateDatabaseQuery/CreateDocumentCollectionQuery producirán llamadas de metadatos al servicio, lo que consume el límite de RU reservadas por el sistema. Estas operaciones deben realizarse raramente.

## <a name="rate-limiting-due-to-transient-service-error"></a>Limitación de tasa debido a un error transitorio del servicio

Este error 429 se devuelve cuando la solicitud encuentra un error de servicio transitorio. El aumento de las RU/s de la base de datos o contenedor no tendrá ningún impacto y no se recomienda.

#### <a name="recommended-solution"></a>Solución recomendada
Vuelva a intentarlo. Si el error persiste durante varios minutos, abra una incidencia de soporte técnico desde [Azure Portal](https://portal.azure.com/).

## <a name="next-steps"></a>Pasos siguientes
* [Supervisión del consumo normalizado de RU/s](monitor-normalized-request-units.md) de la base de datos o el contenedor.
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al utilizar el SDK de Azure Cosmos DB para .NET.
* Más información sobre las directrices de rendimiento de [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET v2](performance-tips.md).
* [Diagnóstico y solución de problemas](troubleshoot-java-sdk-v4-sql.md) al utilizar el SDK de Azure Cosmos DB para Java v4.
* Obtenga información sobre las instrucciones de rendimiento del [SDK para Java v4](performance-tips-java-sdk-v4-sql.md).