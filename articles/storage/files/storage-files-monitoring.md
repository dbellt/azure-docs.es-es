---
title: Supervisión de Azure Files | Microsoft Docs
description: Obtenga información sobre cómo supervisar el rendimiento y la disponibilidad de Azure Files. Supervise los datos de Azure Files, obtenga información sobre la configuración y analice los datos de métricas y registro.
author: normesta
services: storage
ms.service: storage
ms.subservice: files
ms.topic: conceptual
ms.date: 3/02/2021
ms.author: normesta
ms.reviewer: fryu
ms.custom: monitoring, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 620ee3bc5978da4b274aed9a412679ae0835f0b9
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107759834"
---
# <a name="monitoring-azure-files"></a>Supervisión de Azure Files

Si tiene aplicaciones y procesos empresariales críticos que dependen de recursos de Azure, querrá supervisar esos recursos para su disponibilidad, rendimiento y funcionamiento. En este artículo se describen los datos de supervisión que genera Azure Files y cómo puede usar las características de Azure Monitor para analizar las alertas sobre estos datos.

## <a name="monitor-overview"></a>Información general de supervisión

La página **Información general** de Azure Portal para cada recurso de Azure Files incluye una breve vista del uso de los recursos, como las solicitudes y la facturación por hora. Esta información es útil, pero solo hay disponible una pequeña cantidad de datos de supervisión. Algunos de estos datos se recopilan automáticamente y están disponibles para su análisis en cuanto se crea el recurso. Puede habilitar tipos adicionales de recopilación de datos con cierta configuración adicional.

## <a name="what-is-azure-monitor"></a>¿Qué es Azure Monitor?
Azure Files crea los datos de supervisión mediante [Azure Monitor](../../azure-monitor/overview.md), que es un servicio de supervisión de pila completo en Azure. Azure Monitor proporciona un conjunto completo de características para supervisar los recursos de Azure y los recursos en otras nubes y en el entorno local. 

Comience con el artículo [Supervisión de recursos de Azure con Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md), que describe lo siguiente:

- ¿Qué es Azure Monitor?
- Costos asociados con la supervisión
- Datos de supervisión recopilados en Azure
- Configuración de la recolección de datos
- Herramientas estándar en Azure para analizar datos de supervisión y alertar sobre ellos

Las secciones siguientes complementan este artículo mediante la descripción de los datos específicos que se recopilan de Azure Files. En los ejemplos se muestra cómo configurar la recopilación de datos y analizar estos datos con herramientas de Azure.

## <a name="monitoring-data"></a>Supervisión de datos

Azure Files recopila los mismos tipos de datos de supervisión que otros recursos de Azure, que se describen en [Supervisión de datos de recursos de Azure](../../azure-monitor/essentials/monitor-azure-resource.md#monitoring-data). 

Vea [Referencia de datos de supervisión de Azure Files](storage-files-monitoring-reference.md) para obtener información detallada sobre las métricas y las métricas de registros que crea Azure Files.

Las métricas y registros de Azure Monitor solo admiten cuentas de almacenamiento de Azure Resource Manager. Azure Monitor no admite cuentas de almacenamiento clásicas. Si desea usar las métricas o registros en una cuenta de almacenamiento clásica, es preciso migrar a una cuenta de almacenamiento de Azure Resource Manager. Consulte [Migración a Azure Resource Manager](../../virtual-machines/migration-classic-resource-manager-overview.md).

## <a name="collection-and-routing"></a>Recopilación y enrutamiento

Las métricas de la plataforma y el registro de actividad se recopilan automáticamente, pero se pueden enrutar a otras ubicaciones mediante una configuración de diagnóstico. 

Para recopilar registros de recursos, debe crear una configuración de diagnóstico. Cuando cree la configuración, elija **archivo** como tipo de almacenamiento para habilitar los registros. A continuación, especifique una de las siguientes categorías de operaciones de las que quiera recopilar registros. 

| Category | Descripción |
|:---|:---|
| StorageRead | Operaciones de lectura en objetos. |
| StorageWrite | Operaciones de escritura en objetos. |
| StorageDelete | Operaciones de eliminación en objetos. |

Para obtener la lista de operaciones de SMB y de REST que se registran, vea [Operaciones registradas de almacenamiento y mensajes de estado](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) y [Referencia de datos de supervisión de Azure Files](storage-files-monitoring-reference.md).

## <a name="creating-a-diagnostic-setting"></a>Creación de una configuración de diagnóstico

Puede crear una configuración de diagnóstico mediante Azure Portal, PowerShell, la CLI de Azure o una plantilla de Azure Resource Manager.

> [!NOTE]
> Los registros de Azure Storage en Azure Monitor están en versión preliminar pública, además de estar disponibles para pruebas de versión preliminar en todas las regiones de nube pública. Esta versión preliminar habilita los registros de blobs (que incluye Azure Data Lake Storage Gen2), archivos, colas y tablas. Esta característica está disponible para todas las cuentas de almacenamiento que se crean con el modelo de implementación de Azure Resource Manager. Consulte [Introducción a las cuentas de almacenamiento](../common/storage-account-overview.md).

Para obtener instrucciones generales, consulte [Creación de una configuración de diagnóstico para recopilar registros y métricas de la plataforma en Azure](../../azure-monitor/essentials/diagnostic-settings.md).

### <a name="azure-portal"></a>[Azure Portal](#tab/azure-portal)

1. Inicie sesión en Azure Portal.

2. Vaya a la cuenta de almacenamiento.

3. En la sección **Supervisión**, seleccione **Configuración de diagnóstico (versión preliminar)** .

   > [!div class="mx-imgBorder"]
   > ![Portal: Registros de diagnóstico](media/storage-files-monitoring/diagnostic-logs-settings-pane.png)   

4. Elija **archivo** como tipo de almacenamiento para habilitar los registros.

5. Haga clic en **Agregar configuración de diagnóstico**.

   > [!div class="mx-imgBorder"]
   > ![Portal: registros de recursos; agregar configuración de diagnóstico](media/storage-files-monitoring/diagnostic-logs-settings-pane-2.png)

   Aparecerá la página de **Configuración de diagnóstico**.

   > [!div class="mx-imgBorder"]
   > ![Página de registros de recursos](media/storage-files-monitoring/diagnostic-logs-page.png)

6. En el campo **Nombre** de la página, escriba un nombre para la configuración de registro de recursos. A continuación, seleccione qué operaciones quiere registrar (operaciones de lectura, escritura y eliminación) y dónde quiere que se envíen los registros.

#### <a name="archive-logs-to-a-storage-account"></a>Archivo de registros en una cuenta de almacenamiento

Si opta por archivar los registros en una cuenta de almacenamiento, pagará por el volumen de registros que se envíen a la cuenta de almacenamiento. Para obtener información sobre precios concretos, vea la sección **Registros de la plataforma** de la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

1. Seleccione la casilla **Archivar en una cuenta de almacenamiento** y, a continuación, haga clic en el botón **Configurar**.

   > [!div class="mx-imgBorder"]   
   > ![Almacenamiento de archivos en la página de configuración de diagnóstico ](media/storage-files-monitoring/diagnostic-logs-settings-pane-archive-storage.png)

2. En la lista desplegable de la **Cuenta de almacenamiento**, seleccione la cuenta de almacenamiento en la que quiera archivar los registros, haga clic en el botón **Aceptar** y, a continuación, haga clic en el botón **Guardar**.

   [!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

   > [!NOTE]
   > Antes de elegir una cuenta de almacenamiento como destino de exportación, consulte [Archivar registros de recursos de Azure](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage) para comprender los requisitos previos de la cuenta de almacenamiento.

#### <a name="stream-logs-to-azure-event-hubs"></a>Transmisión de registros a Azure Event Hubs

Si decide transmitir los registros a un centro de eventos, pagará por el volumen de registros que se envíen al centro de eventos. Para obtener información sobre precios concretos, vea la sección **Registros de la plataforma** de la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

1. Seleccione la casilla **Transmitir a un centro de eventos** y haga clic en **Configurar**.

2. En el panel para **Seleccionar un centro de eventos**, elija el espacio de nombres, el nombre y el nombre de la directiva del centro de eventos al que quiere transmitir los registros. 

   > [!div class="mx-imgBorder"]
   > ![Centro de eventos en la página de configuración de diagnóstico](media/storage-files-monitoring/diagnostic-logs-settings-pane-event-hub.png)

3. Haga clic en el botón **Aceptar** y, a continuación, en el botón **Guardar**.

#### <a name="send-logs-to-azure-log-analytics"></a>Envío de registros a Azure Log Analytics

1. Seleccione la casilla **Enviar a Log Analytics**, seleccione un área de trabajo de Log Analytics y, a continuación, haga clic en el botón **Guardar**.

   > [!div class="mx-imgBorder"]   
   > ![Instancia de Log Analytics de la página de configuración de diagnóstico](media/storage-files-monitoring/diagnostic-logs-settings-pane-log-analytics.png)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Abra una ventana de comandos de Windows PowerShell e inicie sesión en su suscripción de Azure con el comando `Connect-AzAccount`. A continuación, siga las instrucciones que aparecen en pantalla.

   ```powershell
   Connect-AzAccount
   ```

2. Establezca la suscripción activa como la suscripción de la cuenta de almacenamiento en la que quiere habilitar el registro.

   ```powershell
   Set-AzContext -SubscriptionId <subscription-id>
   ```

#### <a name="archive-logs-to-a-storage-account"></a>Archivo de registros en una cuenta de almacenamiento

Si opta por archivar los registros en una cuenta de almacenamiento, pagará por el volumen de registros que se envíen a la cuenta de almacenamiento. Para obtener información sobre precios concretos, vea la sección **Registros de la plataforma** de la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Habilite los registros mediante el cmdlet de PowerShell [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) junto con el parámetro `StorageAccountId`.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -StorageAccountId <storage-account-resource-id> -Enabled $true -Category <operations-to-log> 
```

Reemplace el marcador de posición `<storage-service-resource--id>` en este fragmento de código por el id. de recursos del servicio de archivos de Azure. Para encontrar el identificador de recurso en Azure Portal, abra la página **Propiedades** de la cuenta de almacenamiento.

Puede usar `StorageRead`, `StorageWrite` y `StorageDelete` en el parámetro **Category**.

[!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

Este es un ejemplo:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/fileServices/default -StorageAccountId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount -Enabled $true -Category StorageWrite,StorageDelete`

Para obtener una descripción de cada parámetro, consulte [Archivo de registros de recursos de Azure mediante Azure PowerShell](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage).

#### <a name="stream-logs-to-an-event-hub"></a>Transmitir registros a un centro de eventos

Si decide transmitir los registros a un centro de eventos, pagará por el volumen de registros que se envíen al centro de eventos. Para obtener información sobre precios concretos, vea la sección **Registros de la plataforma** de la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Habilite los registros mediante el cmdlet de PowerShell [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) con el parámetro `EventHubAuthorizationRuleId`.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -EventHubAuthorizationRuleId <event-hub-namespace-and-key-name> -Enabled $true -Category <operations-to-log> -RetentionEnabled <retention-bool> -RetentionInDays <number-of-days>
```

Este es un ejemplo:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/fileServices/default -EventHubAuthorizationRuleId /subscriptions/20884142-a14v3-4234-5450-08b10c09f4/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhubnamespace/authorizationrules/RootManageSharedAccessKey -Enabled $true -Category StorageDelete`

Para obtener una descripción de cada parámetro, consulte [Transmisión de datos a Event Hubs a través de los cmdlets de PowerShell](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs).

#### <a name="send-logs-to-log-analytics"></a>Envío de registros a Log Analytics

Habilite los registros mediante el cmdlet de PowerShell [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) con el parámetro `WorkspaceId`.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -WorkspaceId <log-analytics-workspace-resource-id> -Enabled $true -Category <operations-to-log> -RetentionEnabled <retention-bool> -RetentionInDays <number-of-days>
```

Este es un ejemplo:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/fileServices/default -WorkspaceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.OperationalInsights/workspaces/my-analytic-workspace -Enabled $true -Category StorageDelete`

Para obtener más información, consulte [Transmisión de registros de recursos de Azure al área de trabajo de Log Analytics en Azure Monitor](../../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace).

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. En primer lugar, abra [Azure Cloud Shell](../../cloud-shell/overview.md) o, si ha [instalado](/cli/azure/install-azure-cli) la CLI de Azure localmente, abra una aplicación de consola de comandos como Windows PowerShell.

2. Si su identidad se asocia a más de una suscripción, establezca su suscripción activa en la suscripción de la cuenta de almacenamiento en la quera habilitar los registros.

   ```azurecli-interactive
   az account set --subscription <subscription-id>
   ```

   Reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

#### <a name="archive-logs-to-a-storage-account"></a>Archivo de registros en una cuenta de almacenamiento

Si opta por archivar los registros en una cuenta de almacenamiento, pagará por el volumen de registros que se envíen a la cuenta de almacenamiento. Para obtener información sobre precios concretos, vea la sección **Registros de la plataforma** de la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Habilite los registros mediante el comando [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az_monitor_diagnostic_settings_create).

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --storage-account <storage-account-name> --resource <storage-service-resource-id> --resource-group <resource-group> --logs '[{"category": <operations>, "enabled": true}]'
```

Reemplace el marcador de posición `<storage-service-resource--id>` en este fragmento de código por el id. de recursos del servicio de Blob Storage. Para encontrar el identificador de recurso en Azure Portal, abra la página **Propiedades** de la cuenta de almacenamiento.

Puede usar `StorageRead`, `StorageWrite` y `StorageDelete` en el valor del parámetro **category**.

[!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

Este es un ejemplo:

`az monitor diagnostic-settings create --name setting1 --storage-account mystorageaccount --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/fileServices/default --resource-group myresourcegroup --logs '[{"category": StorageWrite, "enabled": true}]'`

Para obtener una descripción de cada parámetro, consulte [Archivo de registros de recursos mediante la CLI de Azure](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage).

#### <a name="stream-logs-to-an-event-hub"></a>Transmitir registros a un centro de eventos

Si decide transmitir los registros a un centro de eventos, pagará por el volumen de registros que se envíen al centro de eventos. Para obtener información sobre precios concretos, vea la sección **Registros de la plataforma** de la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Habilite los registros mediante el comando [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az_monitor_diagnostic_settings_create).

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --event-hub <event-hub-name> --event-hub-rule <event-hub-namespace-and-key-name> --resource <storage-account-resource-id> --logs '[{"category": <operations>, "enabled": true "retentionPolicy": {"days": <number-days>, "enabled": <retention-bool}}]'
```

Este es un ejemplo:

`az monitor diagnostic-settings create --name setting1 --event-hub myeventhub --event-hub-rule /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhubnamespace/authorizationrules/RootManageSharedAccessKey --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/fileServices/default --logs '[{"category": StorageDelete, "enabled": true }]'`

Para obtener una descripción de cada parámetro, consulte [Transmisión de datos a Event Hubs a través de la CLI de Azure](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs).

#### <a name="send-logs-to-log-analytics"></a>Envío de registros a Log Analytics

Habilite los registros mediante el comando [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az_monitor_diagnostic_settings_create).

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --workspace <log-analytics-workspace-resource-id> --resource <storage-account-resource-id> --logs '[{"category": <category name>, "enabled": true "retentionPolicy": {"days": <days>, "enabled": <retention-bool}}]'
```

Este es un ejemplo:

`az monitor diagnostic-settings create --name setting1 --workspace /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.OperationalInsights/workspaces/my-analytic-workspace --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/fileServices/default --logs '[{"category": StorageDelete, "enabled": true ]'`

 Para obtener más información, consulte [Transmisión de registros de recursos de Azure al área de trabajo de Log Analytics en Azure Monitor](../../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace).

### <a name="template"></a>[Plantilla](#tab/template)

Para ver una plantilla de Azure Resource Manager que crea una configuración de diagnóstico, consulte [Configuración de diagnóstico para Azure Storage](../../azure-monitor/essentials/resource-manager-diagnostic-settings.md#diagnostic-setting-for-azure-storage).

---

## <a name="analyzing-metrics"></a>Análisis de métricas

Puede analizar las métricas de Azure Storage con métricas de otros servicios de Azure con el Explorador de métricas. Para abrir el Explorador de métricas, elija **Métricas** en el menú **Azure Monitor**. Para más información sobre esta herramienta, consulte [Introducción al Explorador de métricas de Azure](../../azure-monitor/essentials/metrics-getting-started.md). 

Para ver las métricas que admiten dimensiones, puede filtrar las métricas con el valor de dimensión deseado.  Para obtener una lista completa de las dimensiones que admite Azure Storage, consulte [Dimensiones de métricas](storage-files-monitoring-reference.md#metrics-dimensions). Las métricas de Azure Files se encuentran en estos espacios de nombres: 

- Microsoft.Storage/storageAccounts
- Microsoft.Storage/storageAccounts/fileServices

Para obtener una lista de todas las métricas compatibles con Azure Monitor, lo que incluye Azure Files, vea [Métricas compatibles con Azure Monitor](../../azure-monitor/essentials/metrics-supported.md#microsoftstoragestorageaccountsfileservices).

### <a name="accessing-metrics"></a>Acceso a métricas

> [!TIP]
> Para ver ejemplos de la CLI de Azure o .NET, elija las pestañas correspondientes a continuación.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

#### <a name="list-the-metric-definition"></a>Enumeración de la definición de métricas

Puede mostrar la definición de la métrica de la cuenta de almacenamiento o el servicio Azure Files. Use el cmdlet [Get-AzMetricDefinition](/powershell/module/az.monitor/get-azmetricdefinition).

En este ejemplo, reemplace el marcador de posición `<resource-ID>` por el id. de recurso de la cuenta de almacenamiento completa o el del servicio Azure Files.  Puede encontrar estos identificadores de recursos en las páginas **Propiedades** de su cuenta de almacenamiento en Azure Portal.

```powershell
   $resourceId = "<resource-ID>"
   Get-AzMetricDefinition -ResourceId $resourceId
```

#### <a name="reading-metric-values"></a>Lectura de valores de métricas

Puede leer valores de métricas de nivel de cuenta de la cuenta de almacenamiento o el servicio Azure Files. Use el cmdlet [Get-AzMetric](/powershell/module/Az.Monitor/Get-AzMetric).

```powershell
   $resourceId = "<resource-ID>"
   Get-AzMetric -ResourceId $resourceId -MetricNames "UsedCapacity" -TimeGrain 01:00:00
```

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

#### <a name="list-the-account-level-metric-definition"></a>Enumeración de la definición de métricas a nivel de cuenta

Puede mostrar la definición de la métrica de la cuenta de almacenamiento o el servicio Azure Files. Use el comando [az monitor metrics list-definitions](/cli/azure/monitor/metrics#az_monitor_metrics_list_definitions).
 
En este ejemplo, reemplace el marcador de posición `<resource-ID>` por el id. de recurso de la cuenta de almacenamiento completa o el del servicio Azure Files. Puede encontrar estos identificadores de recursos en las páginas **Propiedades** de su cuenta de almacenamiento en Azure Portal.

```azurecli-interactive
   az monitor metrics list-definitions --resource <resource-ID>
```

#### <a name="read-account-level-metric-values"></a>Lectura de valores de métricas a nivel de cuenta

Puede leer los valores de métricas de la cuenta de almacenamiento o el servicio Azure Files. Use el comando [az monitor metrics list](/cli/azure/monitor/metrics#az_monitor_metrics_list).

```azurecli-interactive
   az monitor metrics list --resource <resource-ID> --metric "UsedCapacity" --interval PT1H
```

### <a name="net"></a>[.NET](#tab/azure-portal)

Azure Monitor proporciona el [SDK de .NET](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/) para leer la definición y los valores de las métricas. El [código de ejemplo](https://azure.microsoft.com/resources/samples/monitor-dotnet-metrics-api/) muestra cómo utilizar el SDK con parámetros diferentes. Debe usar `0.18.0-preview` o una versión posterior para las métricas de almacenamiento.
 
En estos ejemplos, reemplace el marcador de posición `<resource-ID>` por el id. de recurso de la cuenta de almacenamiento completa o del servicio Azure Files. Puede encontrar estos identificadores de recursos en las páginas **Propiedades** de su cuenta de almacenamiento en Azure Portal.

Reemplace la variable `<subscription-ID>` por el identificador de la suscripción. Para obtener una guía sobre cómo obtener los valores para `<tenant-ID>`, `<application-ID>` y `<AccessKey>`, consulta [Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos](../../active-directory/develop/howto-create-service-principal-portal.md). 

#### <a name="list-the-account-level-metric-definition"></a>Enumeración de la definición de métricas a nivel de cuenta

En el ejemplo siguiente se muestra cómo enumerar la definición de una métrica a nivel de cuenta:

```csharp
    public static async Task ListStorageMetricDefinition()
    {
        var resourceId = "<resource-ID>";
        var subscriptionId = "<subscription-ID>";
        var tenantId = "<tenant-ID>";
        var applicationId = "<application-ID>";
        var accessKey = "<AccessKey>";


        MonitorManagementClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;
        IEnumerable<MetricDefinition> metricDefinitions = await readOnlyClient.MetricDefinitions.ListAsync(resourceUri: resourceId, cancellationToken: new CancellationToken());

        foreach (var metricDefinition in metricDefinitions)
        {
            // Enumrate metric definition:
            //    Id
            //    ResourceId
            //    Name
            //    Unit
            //    MetricAvailabilities
            //    PrimaryAggregationType
            //    Dimensions
            //    IsDimensionRequired
        }
    }

```

#### <a name="reading-account-level-metric-values"></a>Lectura de valores de métricas a nivel de cuenta

En el ejemplo siguiente se muestra cómo leer los datos de `UsedCapacity` a nivel de cuenta:

```csharp
    public static async Task ReadStorageMetricValue()
    {
        var resourceId = "<resource-ID>";
        var subscriptionId = "<subscription-ID>";
        var tenantId = "<tenant-ID>";
        var applicationId = "<application-ID>";
        var accessKey = "<AccessKey>";

        MonitorClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;

        Microsoft.Azure.Management.Monitor.Models.Response Response;

        string startDate = DateTime.Now.AddHours(-3).ToUniversalTime().ToString("o");
        string endDate = DateTime.Now.ToUniversalTime().ToString("o");
        string timeSpan = startDate + "/" + endDate;

        Response = await readOnlyClient.Metrics.ListAsync(
            resourceUri: resourceId,
            timespan: timeSpan,
            interval: System.TimeSpan.FromHours(1),
            metricnames: "UsedCapacity",

            aggregation: "Average",
            resultType: ResultType.Data,
            cancellationToken: CancellationToken.None);

        foreach (var metric in Response.Value)
        {
            // Enumrate metric value
            //    Id
            //    Name
            //    Type
            //    Unit
            //    Timeseries
            //        - Data
            //        - Metadatavalues
        }
    }

```

#### <a name="reading-multidimensional-metric-values"></a>Lectura de valores de métricas multidimensionales

En el caso de las métricas multidimensionales, debe definir el filtro de metadatos si desea leer datos de métricas sobre valores de dimensión específicos.

En el ejemplo siguiente se muestra cómo leer datos de métricas sobre una métrica que admite varias dimensiones:

```csharp
    public static async Task ReadStorageMetricValueTest()
    {
        // Resource ID for Azure Files
        var resourceId = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/fileServices/default";
        var subscriptionId = "<subscription-ID}";
        // How to identify Tenant ID, Application ID and Access Key: https://azure.microsoft.com/documentation/articles/resource-group-create-service-principal-portal/
        var tenantId = "<tenant-ID>";
        var applicationId = "<application-ID>";
        var accessKey = "<AccessKey>";

        MonitorManagementClient readOnlyClient = AuthenticateWithReadOnlyClient(tenantId, applicationId, accessKey, subscriptionId).Result;

        Microsoft.Azure.Management.Monitor.Models.Response Response;

        string startDate = DateTime.Now.AddHours(-3).ToUniversalTime().ToString("o");
        string endDate = DateTime.Now.ToUniversalTime().ToString("o");
        string timeSpan = startDate + "/" + endDate;
        // It's applicable to define meta data filter when a metric support dimension
        // More conditions can be added with the 'or' and 'and' operators, example: BlobType eq 'BlockBlob' or BlobType eq 'PageBlob'
        ODataQuery<MetadataValue> odataFilterMetrics = new ODataQuery<MetadataValue>(
            string.Format("BlobType eq '{0}'", "BlockBlob"));

        Response = readOnlyClient.Metrics.List(
                        resourceUri: resourceId,
                        timespan: timeSpan,
                        interval: System.TimeSpan.FromHours(1),
                        metricnames: "BlobCapacity",
                        odataQuery: odataFilterMetrics,
                        aggregation: "Average",
                        resultType: ResultType.Data);

        foreach (var metric in Response.Value)
        {
            //Enumrate metric value
            //    Id
            //    Name
            //    Type
            //    Unit
            //    Timeseries
            //        - Data
            //        - Metadatavalues
        }
    }

```

# <a name="template"></a>[Plantilla](#tab/template)

N/D

---

## <a name="analyzing-logs"></a>Análisis de datos

Puede acceder a los registros de los recursos como blob en una cuenta de almacenamiento, como datos de eventos o a través de consultas de Log Analytics.

Para obtener la lista de operaciones de SMB y de REST que se registran, vea [Operaciones registradas de almacenamiento y mensajes de estado](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) y [Referencia de datos de supervisión de Azure Files](storage-files-monitoring-reference.md).

> [!NOTE]
> Los registros de Azure Storage en Azure Monitor están en versión preliminar pública, además de estar disponibles para pruebas de versión preliminar en todas las regiones de nube pública. Esta versión preliminar habilita los registros de blobs (lo que incluye Azure Data Lake Storage Gen2), archivos, colas, tablas, cuentas de almacenamiento Premium en cuentas de almacenamiento de uso general v1 y v2. Las cuentas de almacenamiento clásico no se admiten.

Las entradas del registro se crean solo si se presentan solicitudes al punto de conexión de servicio. Por ejemplo, si una cuenta de almacenamiento tiene actividad en el punto de conexión de su archivo, pero no en los puntos de conexión de su tabla o cola, solo se crean los registros correspondientes al servicio del archivo de Azure. Los registros de Azure Storage contienen información detallada sobre las solicitudes correctas y erróneas realizadas a un servicio de almacenamiento. Esta información se puede utilizar para supervisar solicitudes concretas y para diagnosticar problemas con un servicio de almacenamiento. Las solicitudes se registran en función de la mejor opción.

### <a name="log-authenticated-requests"></a>Registro de solicitudes autenticadas

 Se registran los siguientes tipos de solicitudes autenticadas:

- Solicitudes correctas
- Solicitudes erróneas, incluidos errores de tiempo de espera, de limitación, de red, de autorización y de otro tipo
- Solicitudes que usan Kerberos, NTLM o una firma de acceso compartido (SAS), que incluyen tanto las solicitudes correctas como con error
- Solicitudes de datos de análisis (datos de registro clásicos en el contenedor **$logs** y datos de métricas clásicos en las tablas **$metric**)

Las solicitudes que realiza el propio servicio Azure Files, como la creación o eliminación de registros, no se registran. Para obtener una lista completa de solicitudes de SMB y de REST que se registran, vea [Operaciones registradas de almacenamiento y mensajes de estado](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) y [Referencia de datos de supervisión de Azure Files](storage-files-monitoring-reference.md).

### <a name="accessing-logs-in-a-storage-account"></a>Acceso a registros en una cuenta de almacenamiento

Los registros aparecen como blobs almacenados en un contenedor en la cuenta de almacenamiento de destino. Los datos se recopilan y almacenan en un único blob como una carga JSON delimitada por líneas. El nombre del blob sigue la convención de nomenclatura siguiente:

`https://<destination-storage-account>.blob.core.windows.net/insights-logs-<storage-operation>/resourceId=/subscriptions/<subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<source-storage-account>/fileServices/default/y=<year>/m=<month>/d=<day>/h=<hour>/m=<minute>/PT1H.json`

Este es un ejemplo:

`https://mylogstorageaccount.blob.core.windows.net/insights-logs-storagewrite/resourceId=/subscriptions/`<br>`208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/fileServices/default/y=2019/m=07/d=30/h=23/m=12/PT1H.json`

### <a name="accessing-logs-in-an-event-hub"></a>Acceso a los registros en un centro de eventos

Los registros que se envían a un centro de eventos no se almacenan como archivo, pero puede verificar que el centro de eventos ha recibido la información del registro. En Azure Portal, vaya al centro de eventos y verifique que el recuento de **mensajes entrantes** sea mayor que cero. 

![Registros de auditoría](media/storage-files-monitoring/event-hub-log.png)

Puede acceder y leer los datos de registro que se envían al centro de eventos mediante las herramientas de supervisión y administración de eventos y de información de seguridad. Consulte [¿Qué puedo hacer con los datos de supervisión que se envían a mi centro de eventos?](../../azure-monitor/essentials/stream-monitoring-data-event-hubs.md) para obtener más información.

### <a name="accessing-logs-in-a-log-analytics-workspace"></a>Acceso a registros en un área de trabajo de Log Analytics

Puede acceder a los registros que se envían a un área de trabajo Log Analytics mediante consultas de registro de Azure Monitor. Los datos se almacenan en la tabla **StorageFileLogs**. 

Para obtener más información, vea [Tutorial de Log Analytics](../../azure-monitor/logs/log-analytics-tutorial.md).

#### <a name="sample-kusto-queries"></a>Ejemplos de consultas de Kusto

Estas son algunas consultas que puede escribir en la barra **Búsqueda de registros** para ayudarle a supervisar Azure Files. Estas consultas funcionan con el [nuevo lenguaje](../../azure-monitor/logs/log-query-overview.md).

> [!IMPORTANT]
> Al seleccionar **Registros** en el menú de grupo de recursos de la cuenta de almacenamiento, Log Analytics se abre con el ámbito de la consulta establecido en el grupo de recursos actual. Esto significa que las consultas de registro solo incluirán datos de ese grupo de recursos. Si quiere ejecutar una consulta que incluya datos de otros recursos o de otros servicios de Azure, seleccione **Registros** en el menú **Azure Monitor**. Consulte [Ámbito e intervalo de tiempo de una consulta de registro en Log Analytics de Azure Monitor](../../azure-monitor/logs/scope.md) para obtener más información.

Utilice estas consultas para ayudar a supervisar los recursos compartidos de Azure:

- Ver errores de SMB en la última semana

```Kusto
StorageFileLogs
| where Protocol == "SMB" and TimeGenerated >= ago(7d) and StatusCode contains "-"
| sort by StatusCode
```
- Crear un gráfico circular de las operaciones de SMB en la última semana

```Kusto
StorageFileLogs
| where Protocol == "SMB" and TimeGenerated >= ago(7d) 
| summarize count() by OperationName
| sort by count_ desc
| render piechart
```

- Ver los errores de REST en la última semana

```Kusto
StorageFileLogs
| where Protocol == "HTTPS" and TimeGenerated >= ago(7d) and StatusText !contains "Success"
| sort by StatusText asc
```

- Crear un gráfico circular de las operaciones de REST en la última semana

```Kusto
StorageFileLogs
| where Protocol == "HTTPS" and TimeGenerated >= ago(7d) 
| summarize count() by OperationName
| sort by count_ desc
| render piechart
```

Para ver la lista de nombres y descripciones de las columnas de Azure Files, vea [StorageFileLogs](/azure/azure-monitor/reference/tables/storagefilelogs).

Para obtener más información sobre cómo escribir consultas, vea [Tutorial de Log Analytics](../../azure-monitor/logs/log-analytics-tutorial.md).

## <a name="alerts"></a>Alertas

Las alertas de Azure Monitor le informan de forma proactiva cuando se detectan condiciones importantes en los datos que se supervisan. Permiten identificar y solucionar las incidencias en el sistema antes de que los clientes puedan verlos. Puede establecer alertas en [métricas](../../azure-monitor/alerts/alerts-metric-overview.md), [registros](../../azure-monitor/alerts/alerts-unified-log.md) y el [registro de actividad](../../azure-monitor/alerts/activity-log-alerts.md). 

En la tabla siguiente se muestran algunos escenarios de ejemplo que se van a supervisar y la métrica adecuada que se va a usar para la alerta:

| Escenario | Métrica que se va a usar para la alerta |
|-|-|
| El recurso compartido está limitado. | Métrica: Transactions<br>Nombre de la dimensión: Tipo de respuesta <br>Nombre de la dimensión: FileShare (solo recurso compartido de archivos premium) |
| El tamaño del recurso compartido de archivos es del 80 % de la capacidad. | Métrica: Capacidad de File<br>Nombre de la dimensión: FileShare (solo recurso compartido de archivos premium) |
| La salida del recurso compartido de archivos ha superado los 500 GiB en un día. | Métrica: Salida<br>Nombre de la dimensión: FileShare (solo recurso compartido de archivos premium) |

### <a name="how-to-create-alerts-for-azure-files"></a>Procedimientos a fin de crear alertas para Azure Files

1. Vaya a la **cuenta de almacenamiento** en **Azure Portal**. 

2. Haga clic **Alertas** y, a continuación, haga clic en **Nueva alerta de reglas**.

3. Haga clic en **Editar recurso**, seleccione el **tipo de recurso de archivo** y luego haga clic en **Listo**. 

4. Haga clic en **Agregar condición** y especifique la siguiente información para la alerta: 

    - **Métrica**
    - **Nombre de dimensión**
    - **Lógica de alerta**

5. Haga clic en **Add action groups** (Agregar grupos de acciones) y agregue un grupo de acciones (correo electrónico, SMS, etc.) a la alerta, para lo que puede seleccionar un grupo de acciones existente o crear uno nuevo.

6. Rellene los **detalles de la alerta**, como el **nombre de la regla de alertas**, la **descripción** y la **gravedad**.

7. Haga clic en **Crear regla de alerta** para crear la alerta.

> [!NOTE]  
> Si crea una alerta y esta produce demasiados resultados irrelevantes, ajuste el valor del umbral y la lógica de alerta.

### <a name="how-to-create-an-alert-if-a-file-share-is-throttled"></a>Creación de una alerta si un recurso compartido de archivos se limita

1. Vaya a la **cuenta de almacenamiento** en **Azure Portal**.
2. En la sección **Supervisión**, haga clic en **Alertas** y, después, haga clic en **+ Nueva regla de alertas**.
3. Haga clic en **Editar recurso**, seleccione el **tipo de recurso de archivo** para la cuenta de almacenamiento y, a continuación, haga clic en **Listo**. Por ejemplo, si el nombre de la cuenta de almacenamiento es `contoso`, seleccione el recurso `contoso/file`.
4. Haga clic en **Agregar condición** para agregar una condición.
5. Verá una lista de señales admitidas para la cuenta de almacenamiento, seleccione la métrica **Transacciones**.
6. En la hoja **Configurar lógica de señal**, haga clic en la lista desplegable **Nombre de la dimensión** y seleccione **Tipo de respuesta**.
7. Haga clic en la lista desplegable **Valores de dimensión** y seleccione los tipos de respuesta adecuados para el recurso compartido de archivos.

    Para recursos compartidos de archivos estándar, seleccione los siguientes tipos de respuesta:

    - SuccessWithShareIopsThrottling
    - SuccessWithThrottling
    - ClientShareIopsThrottlingError

    Para recursos compartidos de archivos prémium, seleccione los siguientes tipos de respuesta:

    - SuccessWithShareEgressThrottling
    - SuccessWithShareIngressThrottling
    - SuccessWithShareIopsThrottling
    - ClientShareEgressThrottlingError
    - ClientShareIngressThrottlingError
    - ClientShareIopsThrottlingError

   > [!NOTE]
   > Si los tipos de respuesta no aparecen en la lista desplegable **Valores de dimensión**, significa que el recurso no se ha limitado. Para agregar los valores de dimensión, junto a la lista desplegable **Valores de dimensión**, seleccione **Agregar valor personalizado**, escriba el tipo de respuesta (por ejemplo, **SuccessWithThrottling**), seleccione **Aceptar** y repita estos pasos para agregar todos los tipos de respuesta correspondientes para el recurso compartido de archivos.

8. Para **recursos compartidos de archivos prémium**, haga clic en el menú desplegable **Nombre de la dimensión** y seleccione **Recurso compartido de archivos**. En el caso de **recursos compartidos de archivos estándar**, vaya al **paso n.º 10**.

   > [!NOTE]
   > Si el recurso compartido de archivos es estándar, en la dimensión **Recurso compartido de archivos** no se mostrarán los recursos compartidos de archivo porque las métricas por recurso compartido no están disponibles para los recursos compartidos de archivos estándar. Las alertas de limitación de los recursos compartidos de archivos estándar se desencadenarán si algún recurso compartido de archivos de la cuenta de almacenamiento está limitado y la alerta no identificará qué recurso compartido de archivos se ha limitado. Dado que las métricas por recurso compartido no están disponibles para los recursos compartidos de archivos estándar, se recomienda tener un recurso compartido de archivos por cada cuenta de almacenamiento.

9. Haga clic en la lista desplegable **Valores de dimensión** y seleccione los recursos compartidos de archivos en los que desea generar alertas.
10. Defina los **parámetros de alerta** (umbral, operador, granularidad de agregación y frecuencia de evaluación) y haga clic en **Listo**.

    > [!TIP]
    > Si usa un umbral estático, el gráfico de métricas puede ayudar a determinar un valor de umbral razonable si el recurso compartido de archivos se está limitando actualmente. Si usa un umbral dinámico, el gráfico de métricas mostrará los umbrales calculados según los datos recientes.

11. Haga clic en **Add action groups** (Agregar grupos de acciones) para agregar un **grupo de acciones** (correo electrónico, SMS, etc.) a la alerta, para lo que puede seleccionar un grupo de acciones existente o crear uno nuevo.
12. Rellene los **detalles de la alerta**, como el **nombre de la regla de alertas**, la **descripción** y la **gravedad**.
13. Haga clic en **Crear regla de alerta** para crear la alerta.

### <a name="how-to-create-an-alert-if-the-azure-file-share-size-is-80-of-capacity"></a>Procedimientos para crear una alerta si el tamaño del recurso compartido de archivos de Azure es del 80 % de la capacidad

1. Vaya a la **cuenta de almacenamiento** en **Azure Portal**.
2. En la sección **Supervisión**, haga clic en **Alertas** y luego en **+ Nueva regla de alertas**.
3. Haga clic en **Editar recurso**, seleccione el **tipo de recurso de archivo** para la cuenta de almacenamiento y, a continuación, haga clic en **Listo**. Por ejemplo, si el nombre de la cuenta de almacenamiento es `contoso`, seleccione el recurso `contoso/file`.
4. Haga clic en **Agregar condición** para agregar una condición.
5. Verá una lista de señales admitidas para la cuenta de almacenamiento, seleccione la métrica **Capacidad de archivo**.
6. Para **recursos compartidos de archivos prémium**, haga clic en el menú desplegable **Nombre de la dimensión** y seleccione **Recurso compartido de archivos**. En el caso de **recursos compartidos de archivos estándar**, vaya al **paso n.º 8**.

   > [!NOTE]
   > Si el recurso compartido de archivos es estándar, en la dimensión **Recurso compartido de archivos** no se mostrarán los recursos compartidos de archivo porque las métricas por recurso compartido no están disponibles para los recursos compartidos de archivos estándar. Las alertas de recursos compartidos de archivos estándar se basan en todos los recursos compartidos de archivos de la cuenta de almacenamiento. Dado que las métricas por recurso compartido no están disponibles para los recursos compartidos de archivos estándar, se recomienda tener un recurso compartido de archivos por cada cuenta de almacenamiento.

7. Haga clic en la lista desplegable **Valores de dimensión** y seleccione los recursos compartidos de archivos en los que desea generar alertas.
8. Escriba el **Valor de umbral** (en bytes). Por ejemplo, si el tamaño del recurso compartido de archivos es 100 TiB y quiere recibir una alerta cuando su tamaño sea el 80 % de la capacidad, el valor de umbral en bytes es 87960930222080.
9. Defina los demás **parámetros de alerta** (granularidad de agregación y frecuencia de evaluación) y haga clic en **Listo**.
10. Haga clic en **Add action groups** (Agregar grupos de acciones) para agregar un **grupo de acciones** (correo electrónico, SMS, etc.) a la alerta, para lo que puede seleccionar un grupo de acciones existente o crear uno nuevo.
11. Rellene los **detalles de la alerta**, como el **nombre de la regla de alertas**, la **descripción** y la **gravedad**.
12. Haga clic en **Crear regla de alerta** para crear la alerta.

### <a name="how-to-create-an-alert-if-the-azure-file-share-egress-has-exceeded-500-gib-in-a-day"></a>Procedimientos para crear una alerta si la salida del recurso compartido de archivos de Azure ha superado los 500 GiB en un día

1. Vaya a la **cuenta de almacenamiento** en **Azure Portal**.
2. En la sección Supervisión, haga clic en **Alertas** y, después, haga clic en **+ Nueva regla de alertas**.
3. Haga clic en **Editar recurso**, seleccione el **tipo de recurso de archivo** para la cuenta de almacenamiento y, a continuación, haga clic en **Listo**. Por ejemplo, si el nombre de la cuenta de almacenamiento es contoso, seleccione el recurso contoso/archivo.
4. Haga clic en **Agregar condición** para agregar una condición.
5. Verá una lista de señales admitidas para la cuenta de almacenamiento, seleccione la métrica **Salida**.
6. Para **recursos compartidos de archivos prémium**, haga clic en el menú desplegable **Nombre de la dimensión** y seleccione **Recurso compartido de archivos**. En el caso de **recursos compartidos de archivos estándar**, vaya al **paso n.º 8**.

   > [!NOTE]
   > Si el recurso compartido de archivos es estándar, en la dimensión **Recurso compartido de archivos** no se mostrarán los recursos compartidos de archivo porque las métricas por recurso compartido no están disponibles para los recursos compartidos de archivos estándar. Las alertas de recursos compartidos de archivos estándar se basan en todos los recursos compartidos de archivos de la cuenta de almacenamiento. Dado que las métricas por recurso compartido no están disponibles para los recursos compartidos de archivos estándar, se recomienda tener un recurso compartido de archivos por cada cuenta de almacenamiento.

7. Haga clic en la lista desplegable **Valores de dimensión** y seleccione los recursos compartidos de archivos en los que desea generar alertas.
8. En el Umbral de valor, escriba **536870912000** bytes. 
9. Haga clic en la lista desplegable **Granularidad de agregación** y seleccione **24 horas**.
10. Seleccione la **Frecuencia de evaluación** y **haga clic en Listo**.
11. Haga clic en **Add action groups** (Agregar grupos de acciones) para agregar un **grupo de acciones** (correo electrónico, SMS, etc.) a la alerta, para lo que puede seleccionar un grupo de acciones existente o crear uno nuevo.
12. Rellene los **detalles de la alerta**, como el **nombre de la regla de alertas**, la **descripción** y la **gravedad**.
13. Haga clic en **Crear regla de alerta** para crear la alerta.

## <a name="next-steps"></a>Pasos siguientes

- [Referencia de datos de supervisión de Azure Files](storage-files-monitoring-reference.md)
- [Supervisión de recursos de Azure con Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md)
- [Migración de las métricas de Azure Storage](../common/storage-metrics-migration.md).
- [Planeamiento de una implementación de Azure Files](./storage-files-planning.md)
- [Implementación de Azure Files](./storage-how-to-create-file-share.md)
- [Solución de problemas de Azure Files en Windows](./storage-troubleshoot-windows-file-connection-problems.md)
- [Solución de problemas de Azure Files en Linux](./storage-troubleshoot-linux-file-connection-problems.md)
