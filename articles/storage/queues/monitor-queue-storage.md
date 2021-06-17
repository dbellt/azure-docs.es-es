---
title: Supervisión de Azure Queue Storage
description: Obtenga información sobre cómo supervisar el rendimiento y la disponibilidad de Azure Queue Storage. Supervise los datos de Azure Queue Storage, obtenga información sobre la configuración y analice los datos de métricas y registro.
author: normesta
services: storage
ms.author: normesta
ms.reviewer: fryu
ms.date: 10/26/2020
ms.topic: conceptual
ms.service: storage
ms.subservice: queues
ms.custom: monitoring, devx-track-csharp, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 43cb88f42ff0d61cb3f3e5f4e5475bf51e737457
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110664348"
---
# <a name="monitoring-azure-queue-storage"></a>Supervisión de Azure Queue Storage

Si tiene aplicaciones y procesos empresariales críticos que dependen de recursos de Azure, querrá supervisar esos recursos para su disponibilidad, rendimiento y funcionamiento. En este artículo se describen los datos de supervisión que genera Azure Queue Storage y cómo puede usar las características de Azure Monitor para analizar las alertas sobre estos datos.

> [!NOTE]
> Los registros de Azure Storage en Azure Monitor están en versión preliminar pública, además de estar disponibles para pruebas de versión preliminar en todas las regiones de nube pública. Esta versión preliminar habilita los registros de blobs (que incluye Azure Data Lake Storage Gen2), archivos, colas y tablas. Esta característica está disponible para todas las cuentas de almacenamiento que se crean con el modelo de implementación de Azure Resource Manager. Consulte [Introducción a las cuentas de almacenamiento](../common/storage-account-overview.md).

## <a name="monitor-overview"></a>Información general de supervisión

En la página **Información general** de Azure Portal de cada recurso de Queue Storage se incluye una breve vista del uso del recurso, como las solicitudes y la facturación por hora. Esta información es útil, pero solo hay disponible una pequeña cantidad de datos de supervisión. Algunos de estos datos se recopilan automáticamente y están disponibles para su análisis en cuanto se crea el recurso. Puede habilitar tipos adicionales de recopilación de datos con cierta configuración adicional.

## <a name="what-is-azure-monitor"></a>¿Qué es Azure Monitor?

Azure Queue Storage crea los datos de supervisión mediante [Azure Monitor](../../azure-monitor/overview.md), que es un servicio de supervisión de pila completo en Azure. Azure Monitor proporciona un conjunto completo de características para supervisar los recursos de Azure así como los de otras nubes y en el entorno local.

Comience con [Supervisión de recursos de Azure con Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md), donde se describe lo siguiente:

- ¿Qué es Azure Monitor?
- Costos asociados con la supervisión
- Datos de supervisión recopilados en Azure
- Configuración de la recolección de datos
- Herramientas estándar en Azure para analizar datos de supervisión y alertar sobre ellos

Las secciones siguientes complementan este artículo mediante la descripción de los datos específicos que se recopilan de Azure Storage. En los ejemplos se muestra cómo configurar la recopilación de datos y analizar estos datos con herramientas de Azure.

## <a name="monitoring-data"></a>Supervisión de datos

Azure Queue Storage recopila los mismos tipos de datos de supervisión que otros recursos de Azure, que se describen en [Supervisión de datos de recursos de Azure](../../azure-monitor/essentials/monitor-azure-resource.md#monitoring-data).

Vea [Referencia de datos de supervisión de Azure Queue Storage](monitor-queue-storage-reference.md) para obtener información detallada sobre las métricas y las métricas de registros que crea Azure Queue Storage.

Las métricas y registros de Azure Monitor solo admiten cuentas de almacenamiento de Azure Resource Manager. Azure Monitor no admite cuentas de almacenamiento clásicas. Si desea usar las métricas o registros en una cuenta de almacenamiento clásica, es preciso migrar a una cuenta de almacenamiento de Azure Resource Manager. Consulte [Migración a Azure Resource Manager](../../virtual-machines/migration-classic-resource-manager-overview.md).

Si lo desea, puede seguir usando las métricas y registros clásicos. De hecho, las métricas y registros clásicos están disponibles en paralelo a las métricas y registros en Azure Monitor. La compatibilidad sigue siendo la misma hasta que Azure Storage finalice el servicio en las métricas y registros heredados.

## <a name="collection-and-routing"></a>Recopilación y enrutamiento

Las métricas de la plataforma y el registro de actividad se recopilan automáticamente, pero se pueden enrutar a otras ubicaciones mediante una configuración de diagnóstico.

Para recopilar registros de recursos, debe crear una configuración de diagnóstico. Cuando cree la configuración, elija **cola** como tipo de almacenamiento para habilitar los registros. A continuación, especifique una de las siguientes categorías para las operaciones de las que quiera recopilar registros.

| Category | Descripción |
|:---|:---|
| **StorageRead** | Operaciones de lectura en objetos. |
| **StorageWrite** | Operaciones de escritura en objetos. |
| **StorageDelete** | Operaciones de eliminación en objetos. |

## <a name="creating-a-diagnostic-setting"></a>Creación de una configuración de diagnóstico

Puede crear una configuración de diagnóstico mediante Azure Portal, PowerShell, la CLI de Azure o una plantilla de Azure Resource Manager.

Para obtener instrucciones generales, consulte [Creación de una configuración de diagnóstico para recopilar registros y métricas de la plataforma en Azure](../../azure-monitor/essentials/diagnostic-settings.md).

> [!NOTE]
> Los registros de Azure Storage en Azure Monitor están en versión preliminar pública, además de estar disponibles para pruebas de versión preliminar en todas las regiones de nube pública. Esta versión preliminar habilita los registros de blobs (que incluye Azure Data Lake Storage Gen2), archivos, colas y tablas. Esta característica está disponible para todas las cuentas de almacenamiento que se crean con el modelo de implementación de Azure Resource Manager. Consulte [Introducción a las cuentas de almacenamiento](../common/storage-account-overview.md).

### <a name="azure-portal"></a>[Azure Portal](#tab/azure-portal)

1. Inicie sesión en Azure Portal.

2. Vaya a la cuenta de almacenamiento.

3. En la sección **Supervisión**, seleccione **Configuración de diagnóstico (versión preliminar)** .

   > [!div class="mx-imgBorder"]
   > ![Portal: Registros de diagnóstico](media/monitor-queue-storage/diagnostic-logs-settings-pane.png)

4. Elija **cola** como tipo de almacenamiento al que habilitará los registros.

5. Haga clic en **Agregar configuración de diagnóstico**.

   > [!div class="mx-imgBorder"]
   > ![Portal: registros de recursos; agregar configuración de diagnóstico](media/monitor-queue-storage/diagnostic-logs-settings-pane-2.png)

   Aparecerá la página de **Configuración de diagnóstico**.

   > [!div class="mx-imgBorder"]
   > ![Página de registros de recursos](media/monitor-queue-storage/diagnostic-logs-page.png)

6. En el campo **Nombre** de la página, escriba un nombre para esta configuración de registro de recursos. A continuación, seleccione qué operaciones quiere registrar (operaciones de lectura, escritura y eliminación) y dónde quiere que se envíen los registros.

#### <a name="archive-logs-to-a-storage-account"></a>Archivo de registros en una cuenta de almacenamiento

Si opta por archivar los registros en una cuenta de almacenamiento, pagará por el volumen de registros que se envíen a la cuenta de almacenamiento. Para obtener información sobre precios concretos, vea la sección **Registros de la plataforma** de la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

1. Active la casilla **Archivar en una cuenta de almacenamiento** y, después, seleccione el botón **Configurar**.

   > [!div class="mx-imgBorder"]
   > ![Almacenamiento de archivos en la página de configuración de diagnóstico ](media/monitor-queue-storage/diagnostic-logs-settings-pane-archive-storage.png)

2. En la lista desplegable de la **Cuenta de almacenamiento**, seleccione la cuenta de almacenamiento en la que quiera archivar los registros, haga clic en el botón **Aceptar** y, a continuación, seleccione en el botón **Guardar**.

   [!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

   > [!NOTE]
   > Antes de elegir una cuenta de almacenamiento como destino de exportación, consulte [Archivo de registros de recursos de Azure](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage) para comprender los requisitos previos de la cuenta de almacenamiento.

#### <a name="stream-logs-to-azure-event-hubs"></a>Transmisión de registros a Azure Event Hubs

Si decide transmitir los registros a un centro de eventos, pagará por el volumen de registros que se envíen al centro de eventos. Para obtener información sobre precios concretos, vea la sección **Registros de la plataforma** de la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

1. Active la casilla **Transmitir a un centro de eventos** y, luego, seleccione el botón **Configurar**.

2. En el panel para **Seleccionar un centro de eventos**, elija el espacio de nombres, el nombre y el nombre de la directiva del centro de eventos al que quiere transmitir los registros.

   > [!div class="mx-imgBorder"]
   > ![Centro de eventos en la página de configuración de diagnóstico](media/monitor-queue-storage/diagnostic-logs-settings-pane-event-hub.png)

3. Haga clic en el botón **Aceptar** y, a continuación, seleccione el botón **Guardar**.

#### <a name="send-logs-to-azure-log-analytics"></a>Envío de registros a Azure Log Analytics

1. Active la casilla **Enviar a Log Analytics**, seleccione un área de trabajo de Log Analytics y después el botón **Guardar**.

   > [!div class="mx-imgBorder"]
   > ![Instancia de Log Analytics en la página de configuración de diagnóstico](media/monitor-queue-storage/diagnostic-logs-settings-pane-log-analytics.png)

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

Habilite los registros mediante el uso del cmdlet de PowerShell [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) junto con el parámetro `StorageAccountId`.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -StorageAccountId <storage-account-resource-id> -Enabled $true -Category <operations-to-log>
```

Reemplace el marcador de posición `<storage-service-resource--id>` en este fragmento de código por el id. de recurso de la cola. Para encontrar el identificador de recurso en Azure Portal, abra la página **Propiedades** de la cuenta de almacenamiento.

Puede usar `StorageRead`, `StorageWrite` y `StorageDelete` en el parámetro **Category**.

[!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

Este es un ejemplo:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/queueServices/default -StorageAccountId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount -Enabled $true -Category StorageWrite,StorageDelete`

Para obtener una descripción de cada parámetro, vea [Archivo de registros de recursos de Azure mediante Azure PowerShell](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage).

#### <a name="stream-logs-to-an-event-hub"></a>Transmitir registros a un centro de eventos

Si decide transmitir los registros a un centro de eventos, pagará por el volumen de registros que se envíen al centro de eventos. Para obtener información sobre precios concretos, vea la sección **Registros de la plataforma** de la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Habilite los registros mediante el cmdlet de PowerShell [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) con el parámetro `EventHubAuthorizationRuleId`.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -EventHubAuthorizationRuleId <event-hub-namespace-and-key-name> -Enabled $true -Category <operations-to-log> -RetentionEnabled <retention-bool> -RetentionInDays <number-of-days>
```

Este es un ejemplo:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/queueServices/default -EventHubAuthorizationRuleId /subscriptions/20884142-a14v3-4234-5450-08b10c09f4/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhubnamespace/authorizationrules/RootManageSharedAccessKey -Enabled $true -Category StorageDelete`

Para obtener una descripción de cada parámetro, vea [Transmisión de datos a Event Hubs mediante los cmdlets de PowerShell](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs).

#### <a name="send-logs-to-log-analytics"></a>Envío de registros a Log Analytics

Habilite los registros mediante el cmdlet de PowerShell [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) con el parámetro `WorkspaceId`.

```powershell
Set-AzDiagnosticSetting -ResourceId <storage-service-resource-id> -WorkspaceId <log-analytics-workspace-resource-id> -Enabled $true -Category <operations-to-log> -RetentionEnabled <retention-bool> -RetentionInDays <number-of-days>
```

Este es un ejemplo:

`Set-AzDiagnosticSetting -ResourceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/queueServices/default -WorkspaceId /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.OperationalInsights/workspaces/my-analytic-workspace -Enabled $true -Category StorageDelete`

Para obtener más información, vea [Transmisión de registros de recursos de Azure al área de trabajo de Log Analytics en Azure Monitor](../../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace).

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. En primer lugar, abra [Azure Cloud Shell](../../cloud-shell/overview.md) o, si ha [instalado la CLI de Azure](/cli/azure/install-azure-cli) localmente, abra una aplicación de consola de comandos como PowerShell.

2. Si su identidad se asocia a más de una suscripción, establezca su suscripción activa en la suscripción de la cuenta de almacenamiento en la quera habilitar los registros.

   ```azurecli-interactive
   az account set --subscription <subscription-id>
   ```

   Reemplace el valor de marcador de posición `<subscription-id>` por el identificador de la suscripción.

#### <a name="archive-logs-to-a-storage-account"></a>Archivo de registros en una cuenta de almacenamiento

Si opta por archivar los registros en una cuenta de almacenamiento, pagará por el volumen de registros que se envíen a la cuenta de almacenamiento. Para obtener información sobre precios concretos, vea la sección **Registros de la plataforma** de la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Habilite los registros mediante el comando [`az monitor diagnostic-settings create`](/cli/azure/monitor/diagnostic-settings#az_monitor_diagnostic_settings_create).

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --storage-account <storage-account-name> --resource <storage-service-resource-id> --resource-group <resource-group> --logs '[{"category": <operations>, "enabled": true}]'
```

Reemplace el marcador de posición `<storage-service-resource--id>` en este fragmento de código por el id. de recurso de la cola. Para encontrar el identificador de recurso en Azure Portal, abra la página **Propiedades** de la cuenta de almacenamiento.

Puede usar `StorageRead`, `StorageWrite` y `StorageDelete` en el valor del parámetro `category`.

[!INCLUDE [no retention policy](../../../includes/azure-storage-logs-retention-policy.md)]

Este es un ejemplo:

`az monitor diagnostic-settings create --name setting1 --storage-account mystorageaccount --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/queueServices/default --resource-group myresourcegroup --logs '[{"category": StorageWrite, "enabled": true}]'`

Para obtener una descripción de cada parámetro, vea [Archivo de registros de recursos mediante la CLI de Azure](../../azure-monitor/essentials/resource-logs.md#send-to-azure-storage).

#### <a name="stream-logs-to-an-event-hub"></a>Transmitir registros a un centro de eventos

Si decide transmitir los registros a un centro de eventos, pagará por el volumen de registros que se envíen al centro de eventos. Para obtener información sobre precios concretos, vea la sección **Registros de la plataforma** de la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/#platform-logs).

Habilite los registros mediante el comando [`az monitor diagnostic-settings create`](/cli/azure/monitor/diagnostic-settings#az_monitor_diagnostic_settings_create).

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --event-hub <event-hub-name> --event-hub-rule <event-hub-namespace-and-key-name> --resource <storage-account-resource-id> --logs '[{"category": <operations>, "enabled": true "retentionPolicy": {"days": <number-days>, "enabled": <retention-bool}}]'
```

Este es un ejemplo:

`az monitor diagnostic-settings create --name setting1 --event-hub myeventhub --event-hub-rule /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhubnamespace/authorizationrules/RootManageSharedAccessKey --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/queueServices/default --logs '[{"category": StorageDelete, "enabled": true }]'`

Para obtener una descripción de cada parámetro, vea [Transmisión de datos a Event Hubs mediante la CLI de Azure](../../azure-monitor/essentials/resource-logs.md#send-to-azure-event-hubs).

#### <a name="send-logs-to-log-analytics"></a>Envío de registros a Log Analytics

Habilite los registros mediante el comando [`az monitor diagnostic-settings create`](/cli/azure/monitor/diagnostic-settings#az_monitor_diagnostic_settings_create).

```azurecli-interactive
az monitor diagnostic-settings create --name <setting-name> --workspace <log-analytics-workspace-resource-id> --resource <storage-account-resource-id> --logs '[{"category": <category name>, "enabled": true "retentionPolicy": {"days": <days>, "enabled": <retention-bool}}]'
```

Este es un ejemplo:

`az monitor diagnostic-settings create --name setting1 --workspace /subscriptions/208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.OperationalInsights/workspaces/my-analytic-workspace --resource /subscriptions/938841be-a40c-4bf4-9210-08bcf06c09f9/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/myloggingstorageaccount/queueServices/default --logs '[{"category": StorageDelete, "enabled": true ]'`

 Para obtener más información, vea [Transmisión de registros de recursos de Azure al área de trabajo de Log Analytics en Azure Monitor](../../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace).

# <a name="template"></a>[Plantilla](#tab/template)

Para ver una plantilla de Azure Resource Manager que crea una configuración de diagnóstico, consulte [Configuración de diagnóstico para Azure Storage](../../azure-monitor/essentials/resource-manager-diagnostic-settings.md#diagnostic-setting-for-azure-storage).

---

## <a name="analyzing-metrics"></a>Análisis de métricas

Puede analizar las métricas de Azure Storage con métricas de otros servicios de Azure con el Explorador de métricas de Azure. Para abrir el Explorador de métricas, elija **Métricas** en el menú **Azure Monitor**. Para más información sobre esta herramienta, consulte [Introducción al Explorador de métricas de Azure](../../azure-monitor/essentials/metrics-getting-started.md).

En este ejemplo se muestra cómo ver **Transacciones** a nivel de cuenta.

![Captura de pantalla de acceso a métricas desde Azure Portal](./media/monitor-queue-storage/access-metrics-portal.png)

Para ver las métricas que admiten dimensiones, puede filtrar las métricas con el valor de dimensión deseado. En este ejemplo se muestra cómo ver **Transacciones** en el nivel de cuenta sobre una operación específica mediante la selección de los valores de la dimensión **Nombre de API**.

![Captura de pantalla de acceso a métricas con dimensión desde Azure Portal](./media/monitor-queue-storage/access-metrics-portal-with-dimension.png)

Para obtener una lista completa de las dimensiones que admite Azure Storage, consulte [Dimensiones de métricas](monitor-queue-storage-reference.md#metrics-dimensions).

Las métricas de Azure Queue Storage están en estos espacios de nombres:

- Microsoft.Storage/storageAccounts
- Microsoft.Storage/storageAccounts/queueServices

Para obtener una lista de todas las métricas compatibles de Azure Monitor, lo que incluye Azure Queue Storage, vea [Métricas compatibles con Azure Monitor](../../azure-monitor/essentials/metrics-supported.md).

### <a name="accessing-metrics"></a>Acceso a métricas

> [!TIP]
> Para ver ejemplos de la CLI de Azure o .NET, elija las pestañas correspondientes a continuación.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

#### <a name="list-the-metric-definition"></a>Enumeración de la definición de métricas

Puede mostrar la definición de la métrica de la cuenta de almacenamiento o el servicio Queue Storage. Use el cmdlet [Get-AzMetricDefinition](/powershell/module/az.monitor/get-azmetricdefinition).

En este ejemplo, reemplace el marcador de posición `<resource-ID>` por el identificador de recurso de la cuenta de almacenamiento completa o el de la cola. Puede encontrar estos identificadores de recursos en las páginas **Propiedades** de su cuenta de almacenamiento en Azure Portal.

```powershell
   $resourceId = "<resource-ID>"
   Get-AzMetricDefinition -ResourceId $resourceId
```

#### <a name="reading-metric-values"></a>Lectura de valores de métricas

Puede leer valores de métricas de nivel de cuenta de la cuenta de almacenamiento o el servicio Queue Storage. Use el cmdlet [Get-AzMetric](/powershell/module/az.monitor/get-azmetric).

```powershell
   $resourceId = "<resource-ID>"
   Get-AzMetric -ResourceId $resourceId -MetricNames "UsedCapacity" -TimeGrain 01:00:00
```

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

#### <a name="list-the-account-level-metric-definition"></a>Enumeración de la definición de métricas a nivel de cuenta

Puede mostrar la definición de la métrica de la cuenta de almacenamiento o el servicio Queue Storage. Use el comando [`az monitor metrics list-definitions`](/cli/azure/monitor/metrics#az_monitor_metrics_list_definitions).

En este ejemplo, reemplace el marcador de posición `<resource-ID>` por el identificador de recurso de la cuenta de almacenamiento completa o el de la cola. Puede encontrar estos identificadores de recursos en las páginas **Propiedades** de su cuenta de almacenamiento en Azure Portal.

```azurecli-interactive
   az monitor metrics list-definitions --resource <resource-ID>
```

#### <a name="read-account-level-metric-values"></a>Lectura de valores de métricas a nivel de cuenta

Puede leer los valores de métricas de la cuenta de almacenamiento o el servicio Queue Storage. Use el comando [`az monitor metrics list`](/cli/azure/monitor/metrics#az_monitor_metrics_list).

```azurecli-interactive
   az monitor metrics list --resource <resource-ID> --metric "UsedCapacity" --interval PT1H
```

### <a name="net-sdk"></a>[SDK de .NET](#tab/azure-portal)

Azure Monitor proporciona el [SDK de .NET](https://www.nuget.org/packages/microsoft.azure.management.monitor/) para leer la definición y los valores de las métricas. El [código de ejemplo](https://azure.microsoft.com/resources/samples/monitor-dotnet-metrics-api/) muestra cómo utilizar el SDK con parámetros diferentes. Debe usar `0.18.0-preview` o una versión posterior para las métricas de almacenamiento.

En estos ejemplos, reemplace el marcador de posición `<resource-ID>` por el identificador de recurso de la cuenta de almacenamiento completa o la cola. Puede encontrar estos identificadores de recursos en las páginas **Propiedades** de su cuenta de almacenamiento en Azure Portal.

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
        // Resource ID for queue storage
        var resourceId = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/queueServices/default";
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

### <a name="template"></a>[Plantilla](#tab/template)

N/D

---

## <a name="analyzing-logs"></a>Análisis de datos

Puede acceder a los registros de recursos como una cola en una cuenta de almacenamiento, como datos de eventos o mediante consultas de Log Analytics.

Para obtener una referencia detallada de los campos que aparecen en estos registros, vea [Referencia de datos de supervisión de Azure Queue Storage](monitor-queue-storage-reference.md).

> [!NOTE]
> Los registros de Azure Storage en Azure Monitor están en versión preliminar pública, además de estar disponibles para pruebas de versión preliminar en todas las regiones de nube pública. Esta versión preliminar habilita los registros de blobs (lo que incluye Azure Data Lake Storage Gen2), archivos, colas, tablas, cuentas de almacenamiento Premium en cuentas de almacenamiento de uso general v1 y v2. Las cuentas de almacenamiento clásico no se admiten.

Las entradas del registro se crean solo si se presentan solicitudes al punto de conexión de servicio. Por ejemplo, si una cuenta de almacenamiento tiene actividad en el punto de conexión de la cola, pero no en los de tabla o cola, solo se crean registros correspondientes a Queue Storage. Los registros de Azure Storage contienen información detallada sobre las solicitudes correctas y erróneas realizadas a un servicio de almacenamiento. Esta información se puede utilizar para supervisar solicitudes concretas y para diagnosticar problemas con un servicio de almacenamiento. Las solicitudes se registran en función de la mejor opción.

### <a name="log-authenticated-requests"></a>Registro de solicitudes autenticadas

Se registran los siguientes tipos de solicitudes autenticadas:

- Solicitudes correctas
- Solicitudes erróneas, incluidos errores de tiempo de espera, de limitación, de red, de autorización y de otro tipo
- Solicitudes que usan una firma de acceso compartido (SAS) u OAuth, incluidas las solicitudes correctas como con error
- Solicitudes de datos de análisis (datos de registro clásicos en el contenedor **$logs**, y datos de métricas de clase en las tablas **$metric**)

Las solicitudes realizadas por el propio servicio Queue Storage, como la creación o eliminación de registros, no se registran. Para encontrar una lista completa de los datos registrados, consulte [Operaciones y mensajes de estado registrados por Storage](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) y [Formato del registro de Storage](monitor-queue-storage-reference.md).

### <a name="log-anonymous-requests"></a>Registro de solicitudes anónimas

Se registran los siguientes tipos de solicitudes anónimas:

- Solicitudes correctas
- Errores del servidor
- Errores de tiempo de espera para el cliente y el servidor
- Solicitudes `GET` erróneas con el código de error 304 (`Not Modified`)

El resto de las solicitudes anónimas con error no se registran. Para encontrar una lista completa de los datos registrados, consulte [Operaciones y mensajes de estado registrados por Storage](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) y [Formato del registro de Storage](monitor-queue-storage-reference.md).

### <a name="accessing-logs-in-a-storage-account"></a>Acceso a registros en una cuenta de almacenamiento

Los registros aparecen como blobs almacenados en un contenedor en la cuenta de almacenamiento de destino. Los datos se recopilan y almacenan en un único blob como una carga JSON delimitada por líneas. El nombre del blob sigue la convención de nomenclatura siguiente:

`https://<destination-storage-account>.blob.core.windows.net/insights-logs-<storage-operation>/resourceId=/subscriptions/<subscription-ID>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<source-storage-account>/queueServices/default/y=<year>/m=<month>/d=<day>/h=<hour>/m=<minute>/PT1H.json`

Este es un ejemplo:

`https://mylogstorageaccount.blob.core.windows.net/insights-logs-storagewrite/resourceId=/subscriptions/`<br>`208841be-a4v3-4234-9450-08b90c09f4/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/queueServices/default/y=2019/m=07/d=30/h=23/m=12/PT1H.json`

### <a name="accessing-logs-in-an-event-hub"></a>Acceso a los registros en un centro de eventos

Los registros que se envían a un centro de eventos no se almacenan como archivo, pero puede verificar que el centro de eventos ha recibido la información del registro. En Azure Portal, vaya al centro de eventos y verifique que el recuento de `incoming requests` sea mayor que cero.

![Registros de auditoría](media/monitor-queue-storage/event-hub-log.png)

Puede acceder y leer los datos de registro que se envían al centro de eventos mediante las herramientas de supervisión y administración de eventos y de información de seguridad. Consulte [¿Qué puedo hacer con los datos de supervisión que se envían a mi centro de eventos?](../../azure-monitor/essentials/stream-monitoring-data-event-hubs.md#partner-tools-with-azure-monitor-integration) para obtener más información.

### <a name="accessing-logs-in-a-log-analytics-workspace"></a>Acceso a registros en un área de trabajo de Log Analytics

Puede acceder a los registros que se envían a un área de trabajo Log Analytics mediante consultas de registro de Azure Monitor.

Para más información, consulte [Introducción a los análisis de registros de Azure Monitor](../../azure-monitor/logs/log-analytics-tutorial.md).

Los datos se almacenan en la tabla `StorageQueueLogs`.

#### <a name="sample-kusto-queries"></a>Ejemplos de consultas de Kusto

Estas son algunas consultas que puede escribir en la barra **Búsqueda de registros** para facilitar la supervisión de las colas. Estas consultas funcionan con el [nuevo lenguaje](../../azure-monitor/logs/log-query-overview.md).

> [!IMPORTANT]
> Al seleccionar **Registros** en el menú de grupo de recursos de la cuenta de almacenamiento, Log Analytics se abre con el ámbito de la consulta establecido en el grupo de recursos actual. Esto significa que las consultas de registro solo incluirán datos de ese grupo de recursos. Si quiere ejecutar una consulta que incluya datos de otros recursos o de otros servicios de Azure, seleccione **Registros** en el menú **Azure Monitor**. Consulte [Ámbito e intervalo de tiempo de una consulta de registro en Log Analytics de Azure Monitor](../../azure-monitor/logs/scope.md) para obtener más información.

Utilice estas consultas para ayudar a supervisar las cuentas de Azure Storage:

- Para enumerar los 10 errores más comunes en los últimos tres días.

    ```kusto
    StorageQueueLogs
    | where TimeGenerated > ago(3d) and StatusText !contains "Success"
    | summarize count() by StatusText
    | top 10 by count_ desc
    ```

- Para enumerar las 10 operaciones principales que causaron la mayoría de los errores en los últimos tres días.

    ```kusto
    StorageQueueLogs
    | where TimeGenerated > ago(3d) and StatusText !contains "Success"
    | summarize count() by OperationName
    | top 10 by count_ desc
    ```

- Para enumerar las 10 operaciones principales con la mayor latencia de un extremo a otro en los últimos tres días.

    ```kusto
    StorageQueueLogs
    | where TimeGenerated > ago(3d)
    | top 10 by DurationMs desc
    | project TimeGenerated, OperationName, DurationMs, ServerLatencyMs, ClientLatencyMs = DurationMs - ServerLatencyMs
    ```

- Para enumerar todas las operaciones que provocan errores de limitación del lado servidor durante los últimos tres días.

    ```kusto
    StorageQueueLogs
    | where TimeGenerated > ago(3d) and StatusText contains "ServerBusy"
    | project TimeGenerated, OperationName, StatusCode, StatusText
    ```

- Para enumerar todas las solicitudes con acceso anónimo durante los últimos tres días.

    ```kusto
    StorageBlobLogs
    | where TimeGenerated > ago(3d) and AuthenticationType == "Anonymous"
    | project TimeGenerated, OperationName, AuthenticationType, Uri
    ```

- Para crear un gráfico circular de las operaciones usadas en los últimos tres días.

    ```kusto
    StorageQueueLogs
    | where TimeGenerated > ago(3d)
    | summarize count() by OperationName
    | sort by count_ desc
    | render piechart
    ```

## <a name="faq"></a>Preguntas más frecuentes

**¿Admite Azure Storage métricas de discos administrados o no administrados?**

No. Las instancias de proceso admiten las métricas en los discos. Para obtener más información, vea [Métricas por disco para discos administrados y no administrados](https://azure.microsoft.com/blog/per-disk-metrics-managed-disks/).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener una referencia de los registros y las métricas creados por Azure Queue Storage, vea [Referencia de datos de supervisión de Azure Queue Storage](monitor-queue-storage-reference.md).
- Para obtener más información sobre la supervisión de recursos de Azure, consulte [Supervisión de recursos de Azure con Azure Monitor](../../azure-monitor/essentials/monitor-azure-resource.md).
- Para obtener más información sobre la migración de métricas, consulte [Migración de las métricas de Azure Storage](../common/storage-metrics-migration.md).
