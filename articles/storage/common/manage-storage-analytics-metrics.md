---
title: Habilitación y administración de las métricas de Azure Storage Analytics | Microsoft Docs
description: Obtenga información sobre cómo habilitar, editar y ver las métricas de Azure Storage Analytics.
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 01/29/2021
ms.author: normesta
ms.reviewer: fryu
ms.subservice: common
ms.custom: monitoring, devx-track-azurepowershell
ms.openlocfilehash: d9179872562ea048fe713d9535c4ff2994bcd223
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110666798"
---
# <a name="enable-and-manage-azure-storage-analytics-metrics-classic"></a>Habilitación y administración de las métricas de Azure Storage Analytics (clásico)

[Azure Storage Analytics](storage-analytics.md) proporciona métricas para todos los servicios de almacenamiento en relación con los blobs, las colas y las tablas. Puede usar [Azure Portal](https://portal.azure.com) para configurar qué métricas se registran para su cuenta y configurar gráficos que proporcionen representaciones visuales de los datos de las métricas. En este artículo se muestra cómo habilitar y administrar las métricas. Para obtener información sobre cómo habilitar los registros, vea [Habilitación y administración de los registros de Azure Storage Analytics (clásico)](manage-storage-analytics-logs.md).

Se recomienda consultar [Azure Monitor para Storage](../../azure-monitor/insights/storage-insights-overview.md) (versión preliminar). Es una característica de Azure Monitor que ofrece una supervisión completa de las cuentas de Azure Storage, ya que ofrece una vista unificada del rendimiento, la capacidad y la disponibilidad de los servicios de Azure Storage. No es preciso habilitar o configurar nada, y puede ver inmediatamente estas métricas desde los gráficos interactivos predefinidos y otras visualizaciones incluidas.

> [!NOTE]
> Existen costos asociados con el análisis de los datos de supervisión en Azure Portal. Para más información, consulte [Storage Analytics](storage-analytics.md).
>
> Las cuentas de almacenamiento de blobs en bloques de rendimiento prémium no admiten las métricas de Storage Analytics. Si desea ver métricas con cuentas de almacenamiento de blobs en bloques de rendimiento prémium, considere la posibilidad de usar [Métricas de Azure Storage en Azure Monitor](../blobs/monitor-blob-storage.md).
>
> Para obtener orientación exhaustiva sobre el uso de análisis de almacenamiento y otras herramientas para identificar, diagnosticar y solucionar problemas relacionados con Azure Storage, consulte [Supervisión, diagnóstico y solución de problemas de Microsoft Azure Storage](storage-monitoring-diagnosing-troubleshooting.md).
>

<a id="Enable-metrics"></a>

## <a name="enable-metrics"></a>Habilitación de métricas

### <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com), seleccione **Cuentas de almacenamiento** y, después, seleccione el nombre de la cuenta de almacenamiento para abrir el panel de esta.

2. Seleccione **Configuración de diagnóstico (clásica)** en la sección **Supervisión (clásica)** de la hoja de menú.
   
   ![Captura de pantalla que resalta la opción Configuración de diagnóstico (clásica) en la sección Supervisión (clásica).](./media/manage-storage-analytics-metrics/storage-enable-metrics-00.png)

3. Seleccione el **tipo** de datos de métricas para cada **servicio** que desee supervisar y la **directiva de retención** para los datos. También puede deshabilitar la supervisión estableciendo el **estado** en **Desactivado**.

   > [!div class="mx-imgBorder"]
   > ![Configure el registro en Azure Portal.](./media/manage-storage-analytics-logs/enable-diagnostics.png) 

   Para configurar la directiva de retención de datos, mueva el control deslizante **Retención (en días)** o escriba el número de días que se deben retener los datos, entre 1 y 365. El valor predeterminado para las nuevas cuentas de almacenamiento es de siete días. Si no desea configurar una directiva de retención, escriba un cero. Si no existe una directiva de retención, es posible eliminar los datos de supervisión.

   > [!WARNING]
   > Las métricas se almacenan como datos en la cuenta. Los datos de las métricas se pueden acumular en la cuenta a lo largo del tiempo, lo que puede aumentar el costo de almacenamiento. Si necesita datos de métricas solo durante un breve período de tiempo, puede reducir los costos mediante la modificación de la directiva de retención de datos. El sistema eliminará los datos de métricas obsoletos (aquellos anteriores a la directiva de retención de datos). Es recomendable configurar una directiva de retención basada en el tiempo que desee retener los datos de métricas de su cuenta. Vea [Facturación según las métricas de almacenamiento](storage-analytics-metrics.md#billing-on-storage-metrics) para más información.
   >

4. Al finalizar la configuración de la supervisión, seleccione **Guardar**.

Un conjunto predeterminado de métricas se muestra en los gráficos de la hoja **Información general**, así como en la hoja **Métricas (clásica)** . Una vez que ha habilitado las métricas de un servicio, puede tardar una hora hasta que los datos aparezcan en los gráficos. Puede seleccionar **Editar** en cualquier gráfico de métricas para configurar qué métricas se muestran en el gráfico.

Puede deshabilitar la recopilación y el registro de métricas estableciendo el **estado** en **Desactivado**.

> [!NOTE]
> Azure Storage usa [Table Storage](storage-introduction.md#table-storage) para almacenar las métricas para la cuenta de almacenamiento y almacena las métricas en tablas en su cuenta. Para obtener más información, vea: [Cómo se almacenan las métricas](storage-analytics-metrics.md#how-metrics-are-stored).

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Abra una ventana de comando de Windows PowerShell.

2. Inicie sesión en la suscripción a Azure con el comando `Connect-AzAccount` y siga las instrucciones de la pantalla.

   ```powershell
   Connect-AzAccount
   ```

3. Si su identidad está asociada a más de una suscripción, establezca la suscripción activa.

   ```powershell
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

   Reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

5. Obtenga el contexto de la cuenta de almacenamiento que define la cuenta de almacenamiento que desea usar.

   ```powershell
   $storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -AccountName "<storage-account-name>"
   $ctx = $storageAccount.Context
   ```

   * Reemplace el marcador de posición `<resource-group-name>` por el nombre del grupo de recursos.

   * Reemplace el valor de marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento. 

6. Puede usar PowerShell en el equipo local para configurar las métricas de almacenamiento en la cuenta de almacenamiento. Use el cmdlet **Set-AzStorageServiceMetricsProperty** de Azure PowerShell para cambiar la configuración actual. 

   El siguiente comando activa las métricas por minuto para Blob service en su cuenta de almacenamiento con un período de retención establecido en 5 días.

   ```powershell
   Set-AzStorageServiceMetricsProperty -MetricsType Minute -ServiceType Blob -MetricsLevel ServiceAndApi  -RetentionDays 5 -Context $ctx
   ```   

   Este cmdlet utiliza los siguientes parámetros:  

   - **ServiceType**: Los valores posibles son **Blob**, **Queue**, **Table** y **File**.
   - **MetricsType**: Los valores posibles son **Hour** y **Minute**.  
   - **MetricsLevel**: Los valores posibles son:
      - **Ninguna**: desactiva la supervisión.
      - **Servicio**: Recopila métricas, como entrada y salida, disponibilidad, latencia y porcentajes de éxito, que se agregan a Blob service, Queue service, Table service y File service.
      - **ServiceAndApi**: Además de las métricas de servicio, recopila el mismo conjunto de métricas de cada operación de almacenamiento en la API de servicio Azure Storage.

   El comando siguiente recupera el nivel de métricas por hora actual y los días de retención para el servicio BLOB en su cuenta de almacenamiento predeterminada:  

   ```powershell
   Get-AzStorageServiceMetricsProperty -MetricsType Hour -ServiceType Blob -Context $storagecontext.Context
   ```  

   Para obtener información sobre cómo configurar los cmdlets de Azure PowerShell para que funcionen con su suscripción a Azure y cómo seleccionar la cuenta de almacenamiento predeterminada que quiere usar, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).  

### <a name="net-v12-sdk"></a>[SDK de .NET, versión 12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/Monitoring.cs" id="snippet_EnableDiagnosticLogs":::

Para obtener más información sobre cómo usar un lenguaje .NET para configurar métricas de almacenamiento, consulte [Biblioteca de cliente de Azure Storage para .NET](/dotnet/api/overview/azure/storage).  

Para obtener información general sobre cómo configurar las métricas de almacenamiento con la API de REST, consulte [Habilitación y configuración de Storage Analytics](/rest/api/storageservices/Enabling-and-Configuring-Storage-Analytics). 

### <a name="net-v11-sdk"></a>[SDK de .NET, versión 11](#tab/dotnet11)  

```csharp
var storageAccount = CloudStorageAccount.Parse(connStr);  
var queueClient = storageAccount.CreateCloudQueueClient();  
var serviceProperties = queueClient.GetServiceProperties();  

serviceProperties.HourMetrics.MetricsLevel = MetricsLevel.Service;  
serviceProperties.HourMetrics.RetentionDays = 10;  

queueClient.SetServiceProperties(serviceProperties);  
```  

Para obtener más información sobre cómo usar un lenguaje .NET para configurar métricas de almacenamiento, consulte [Biblioteca de cliente de Azure Storage para .NET](/dotnet/api/overview/azure/storage).  

Para obtener información general sobre cómo configurar las métricas de almacenamiento con la API de REST, consulte [Habilitación y configuración de Storage Analytics](/rest/api/storageservices/Enabling-and-Configuring-Storage-Analytics). 

---

<a id="view-metrics"></a>

## <a name="view-metrics-in-a-chart"></a>Visualización de las métricas en un gráfico

Cuando haya configurado las métricas de Storage Analytics para supervisar su cuenta de almacenamiento, Storage Analytics se encargará de registrar las métricas en un conjunto de tablas conocidas en su cuenta de almacenamiento. Puede configurar gráficos para ver las métricas por hora en [Azure Portal](https://portal.azure.com).

Utilice el procedimiento siguiente para elegir las métricas de almacenamiento que desea ver en un gráfico de métricas.

1. Comience por mostrar un gráfico de métricas de almacenamiento en Azure Portal. Puede encontrar los gráficos en la **hoja de la cuenta de almacenamiento** y en la hoja **Métricas (clásica)** .

   En este ejemplo, se usa el siguiente gráfico que aparece en la **hoja de la cuenta de almacenamiento**:

   ![Selección de gráfico en Azure Portal](./media/manage-storage-analytics-metrics/stg-customize-chart-00.png)

2. Haga clic en cualquier lugar dentro del gráfico para modificarlo.

3. Después, seleccione el **Intervalo de tiempo** de la métrica para mostrar en el gráfico y el **servicio** (blob, cola, tabla, archivo) cuya métrica desee mostrar. Aquí se seleccionan las métricas de la semana anterior para mostrar para el servicio Blob:

   ![Selección de intervalo de tiempo y servicio en la hoja Editar gráfico](./media/manage-storage-analytics-metrics/storage-edit-metric-time-range.png)

4. Seleccione las **métricas** individuales que desea que aparezcan en el gráfico y, a continuación, haga clic en **Aceptar**.

   ![Selección de métricas individuales en la hoja Editar gráfico](./media/manage-storage-analytics-metrics/storage-edit-metric-selections.png)

Estas configuraciones del gráfico no afectan a la recopilación, la incorporación o el almacenamiento de los datos de supervisión en la cuenta de almacenamiento.

#### <a name="metrics-availability-in-charts"></a>Disponibilidad de las métricas en los gráficos

La lista de las métricas disponibles cambia en función de qué servicio haya elegido en la lista desplegable y del tipo de unidad del gráfico que está editando. Por ejemplo, puede seleccionar métricas de porcentaje como *PercentNetworkError* y *PercentThrottlingError* solo si está editando un gráfico que muestra las unidades en porcentajes:

![Solicitud de gráfico de porcentaje de errores en Azure Portal](./media/manage-storage-analytics-metrics/stg-customize-chart-04.png)

#### <a name="metrics-resolution"></a>Resolución de métricas

Las métricas seleccionadas en **Diagnóstico** determinan la resolución de las métricas que están disponibles para su cuenta:

* **Agregados**: la supervisión proporciona métricas como entrada/salida, disponibilidad, latencia y porcentajes de éxito. Estas métricas se agregan a los servicios Blob, Cola, Tabla y Archivo.
* **Por API** proporciona una resolución más precisa, con métricas disponibles para operaciones de almacenamiento individuales, además de los agregados de nivel de servicio.

## <a name="download-metrics-to-archive-or-analyze-locally"></a>Descarga de las métricas para archivarlas o analizarlas localmente

Si quiere descargar las métricas para almacenarlas de forma prolongada o para analizarlas localmente, debe usar una herramienta o escribir código para leer las tablas. Las tablas no aparecen si enumera todas las tablas en su cuenta de almacenamiento, pero puede acceder a ellas directamente por su nombre. Muchas herramientas de exploración de almacenamiento reconocen estas tablas y le permiten verlas directamente. Para obtener una lista de herramientas disponibles, consulte [Herramientas de cliente de Azure Storage](./storage-explorers.md).

|Métricas|Nombres de tabla|Notas| 
|-|-|-|  
|Métricas por hora|$MetricsHourPrimaryTransactionsBlob<br /><br /> $MetricsHourPrimaryTransactionsTable<br /><br /> $MetricsHourPrimaryTransactionsQueue<br /><br /> $MetricsHourPrimaryTransactionsFile|En versiones anteriores al 15 de agosto de 2013, estas tablas se denominaban:<br /><br /> $MetricsTransactionsBlob<br /><br /> $MetricsTransactionsTable<br /><br /> $MetricsTransactionsQueue<br /><br /> Las métricas de File service están disponibles a partir de la versión del 5 de abril de 2015.|  
|Métricas por minuto|$MetricsMinutePrimaryTransactionsBlob<br /><br /> $MetricsMinutePrimaryTransactionsTable<br /><br /> $MetricsMinutePrimaryTransactionsQueue<br /><br /> $MetricsMinutePrimaryTransactionsFile|Solo se pueden habilitar con PowerShell o mediante programación.<br /><br /> Las métricas de File service están disponibles a partir de la versión del 5 de abril de 2015.|  
|Capacity|$MetricsCapacityBlob|Solo Blob service.|  

Para obtener información completa de los esquemas para estas tablas, consulte [Esquema de las tablas de métricas de Storage Analytics](/rest/api/storageservices/storage-analytics-metrics-table-schema). Las filas de ejemplo siguientes muestran solo un subconjunto de columnas disponibles, pero ilustran algunas características importantes acerca de la manera en que las métricas de almacenamiento guardan estas métricas:  

|PartitionKey|RowKey|Timestamp|TotalRequests|TotalBillableRequests|TotalIngress|TotalEgress|Disponibilidad|AverageE2ELatency|AverageServerLatency|PercentSuccess| 
|-|-|-|-|-|-|-|-|-|-|-|  
|20140522T1100|user;All|2014-05-22T11:01:16.765025Z|7|7|4003|46801|100|104.4286|6.857143|100|  
|20140522T1100|user;QueryEntities|2014-05-22T11:01:16.764025Z|5|5|2694|45951|100|143.8|7.8|100|  
|20140522T1100|user;QueryEntity|2014-05-22T11:01:16.765025Z|1|1|538|633|100|3|3|100|  
|20140522T1100|user;UpdateEntity|2014-05-22T11:01:16.765025Z|1|1|771|217|100|9|6|100|  

En este ejemplo de datos de métricas por minuto, la clave de partición usa el tiempo con una resolución de minutos. La clave de fila identifica el tipo de información que se almacena en la fila. La información se compone del tipo de acceso y del tipo de solicitud:  

-   El tipo de acceso es **user** o **system**, donde **user** hace referencia a todas las solicitudes de usuario al servicio de almacenamiento, y **system** hace referencia a las solicitudes realizadas por Storage Analytics.  
-   El tipo de solicitud es **all**, en cuyo caso es una línea de resumen, o identifica una API específica como **QueryEntity** o **UpdateEntity**.  

Estos datos de ejemplo muestran todos los registros relativos a un único minuto (a partir de las 11:00 a. m.), por lo que el número de solicitudes **QueryEntities** más el número de solicitudes **QueryEntity** más el número de solicitudes **UpdateEntity** suman siete. Este total se muestra en la fila **user:All**. De forma similar, puede derivar la latencia promedio de un extremo a otro 104,4286 en la fila **user:All** calculando ((143,8 x 5) + 3 + 9):7.  

## <a name="view-metrics-data-programmatically"></a>Visualización de los datos de métricas mediante programación

En la lista siguiente se muestra código C# de ejemplo que accede a las métricas por minuto para un intervalo de minutos y muestra los resultados en una ventana de la consola. En el código de ejemplo se usa la biblioteca de cliente de Azure Storage versión 4.x o posterior, que incluye la clase **CloudAnalyticsClient**, que simplifica el acceso a las tablas de métricas en el almacenamiento. 

> [!NOTE]
> La clase **CloudAnalyticsClient** no se incluye en la biblioteca de cliente de Azure Blob Storage v12 para .NET. El **31 de agosto de 2023**, las métricas de Storage Analytics, también denominadas *métricas clásicas*, se retirarán. Para obtener más información, consulte el [anuncio oficial](https://azure.microsoft.com/updates/azure-storage-classic-metrics-will-be-retired-on-31-august-2023/). Si usa métricas clásicas, se recomienda realizar la transición a las métricas de Azure Monitor antes de esa fecha. 

```csharp
private static void PrintMinuteMetrics(CloudAnalyticsClient analyticsClient, DateTimeOffset startDateTime, DateTimeOffset endDateTime)  
{  
 // Convert the dates to the format used in the PartitionKey.  
 var start = startDateTime.ToUniversalTime().ToString("yyyyMMdd'T'HHmm");  
 var end = endDateTime.ToUniversalTime().ToString("yyyyMMdd'T'HHmm");  

 var services = Enum.GetValues(typeof(StorageService));  
 foreach (StorageService service in services)  
 {  
     Console.WriteLine("Minute Metrics for Service {0} from {1} to {2} UTC", service, start, end);  
     var metricsQuery = analyticsClient.CreateMinuteMetricsQuery(service, StorageLocation.Primary);  
     var t = analyticsClient.GetMinuteMetricsTable(service);  
     var opContext = new OperationContext();  
     var query =  
             from entity in metricsQuery  
             // Note, you can't filter using the entity properties Time, AccessType, or TransactionType  
             // because they are calculated fields in the MetricsEntity class.  
             // The PartitionKey identifies the DataTime of the metrics.  
             where entity.PartitionKey.CompareTo(start) >= 0 && entity.PartitionKey.CompareTo(end) <= 0   
             select entity;  

     // Filter on "user" transactions after fetching the metrics from Azure Table storage.  
     // (StartsWith is not supported using LINQ with Azure Table storage.)  
     var results = query.ToList().Where(m => m.RowKey.StartsWith("user"));  
     var resultString = results.Aggregate(new StringBuilder(), (builder, metrics) => builder.AppendLine(MetricsString(metrics, opContext))).ToString();  
     Console.WriteLine(resultString);  
 }  
}  

private static string MetricsString(MetricsEntity entity, OperationContext opContext)  
{  
 var entityProperties = entity.WriteEntity(opContext);  
 var entityString =  
         string.Format("Time: {0}, ", entity.Time) +  
         string.Format("AccessType: {0}, ", entity.AccessType) +  
         string.Format("TransactionType: {0}, ", entity.TransactionType) +  
         string.Join(",", entityProperties.Select(e => new KeyValuePair<string, string>(e.Key.ToString(), e.Value.PropertyAsObject.ToString())));  
 return entityString;  
}  
```  

<a id="add-metrics-to-dashboard"></a>

## <a name="add-metrics-charts-to-the-portal-dashboard"></a>Adición de gráficos de métricas al panel del portal

Puede agregar gráficos de métricas de Azure Storage a cualquiera de las cuentas de almacenamiento del panel del portal.

1. Seleccione **Editar panel** mientras lo ve en [Azure Portal](https://portal.azure.com).
1. En la **galería de iconos**, seleccione **Buscar iconos por** > **Tipo**.
1. Seleccione **Tipo** > **Cuentas de almacenamiento**.
1. En **Recursos**, seleccione la cuenta de almacenamiento cuyas métricas desea agregar al panel.
1. Seleccione **Categorías** > **Supervisión**.
1. Arrastre y coloque el icono del gráfico en el panel de la métrica que desea mostrar. Repita este paso para todas las métricas que desee que se muestren en el panel. En la siguiente imagen, se resalta el gráfico "Blobs: Número total de solicitudes" como un ejemplo, pero todos los gráficos están disponibles para su ubicación en el panel.

   ![Galería de iconos de Azure Portal](./media/manage-storage-analytics-metrics/storage-customize-dashboard.png)
1. Seleccione la opción **Personalización finalizada** situada cerca de la parte superior del panel cuando haya terminado de agregar los gráficos.

Una vez que haya agregado los gráficos al panel, los puede personalizar aún más tal y como se describe en Personalización de gráficos de métricas.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre Storage Analytics, vea [Storage Analytics](storage-analytics.md) para Storage Analytics.
* [Configure registros de Storage Analytics](manage-storage-analytics-logs.md).
* Obtenga más información sobre el esquema de métricas. Vea [Esquema de las tablas de métricas de Storage Analytics](/rest/api/storageservices/storage-analytics-metrics-table-schema).
