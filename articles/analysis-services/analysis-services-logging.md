---
title: Registro de diagnóstico de Azure Analysis Services | Microsoft Docs
description: Describe cómo configurar el registro para supervisar el servidor de Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8ede7572079b6a54672234cbf9fe1445dafbad7b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107769218"
---
# <a name="setup-diagnostic-logging"></a>Configuración del registro de diagnóstico

Una parte importante de cualquier solución de Analysis Services es la supervisión de cómo se comportan los servidores en cuanto al rendimiento. Azure Analysis Services se integra en Azure Monitor. Con los [registros de recursos de Azure Monitor](../azure-monitor/essentials/platform-logs-overview.md), puede supervisar los registros, enviarlos a [Azure Storage](https://azure.microsoft.com/services/storage/), transmitirlos a [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/) y exportarlos a los [registros de Azure Monitor](../azure-monitor/overview.md).

![Registro de recursos en Storage, Event Hubs o los registros de Azure Monitor](./media/analysis-services-logging/aas-logging-overview.png)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="whats-logged"></a>¿Qué se registra?

Puede seleccionar las categorías **Motor**, **Servicio** y **Métricas**.

### <a name="engine"></a>Motor

Al seleccionar **Motor**, se registran todos los [xEvents](/analysis-services/instances/monitor-analysis-services-with-sql-server-extended-events). No puede seleccionar eventos individuales. 

|Categorías de XEvent |Nombre del evento  |
|---------|---------|
|Auditoría de seguridad    |   Audit Login      |
|Auditoría de seguridad    |   Audit Logout      |
|Auditoría de seguridad    |   Audit Server Starts And Stops      |
|Informes de progreso     |   Progress Report Begin      |
|Informes de progreso     |   Progress Report End      |
|Informes de progreso     |   Progress Report Current      |
|Consultas     |  Query Begin       |
|Consultas     |   Query End      |
|Comandos:     |  Command Begin       |
|Comandos:     |  Command End       |
|Errores y advertencias     |   Error      |
|Descubra     |   Discover End      |
|Notification     |    Notification     |
|Sesión     |  Session Initialize       |
|Bloqueos    |  Deadlock       |
|Procesamiento de consultas     |   VertiPaq SE Query Begin      |
|Procesamiento de consultas     |   VertiPaq SE Query End      |
|Procesamiento de consultas     |   VertiPaq SE Query Cache Match      |
|Procesamiento de consultas     |   Direct Query Begin      |
|Procesamiento de consultas     |  Direct Query End       |

### <a name="service"></a>Servicio

|Nombre de la operación  |Ocurre cuando  |
|---------|---------|
|ResumeServer     |    Se reanuda un servidor     |
|SuspendServer    |   Se pausa un servidor      |
|DeleteServer     |    Eliminación de un servidor     |
|RestartServer    |     El usuario reinicia un servidor a través de SSMS o PowerShell    |
|GetServerLogFiles    |    El usuario exporta el registro de servidor a través de PowerShell     |
|ExportModel     |   El usuario exporta un modelo en el portal con Abrir en Visual Studio     |

### <a name="all-metrics"></a>Todas las métricas

La categoría Métricas registra las mismas [métricas de servidor](analysis-services-monitor.md#server-metrics) que se muestran en AzureMetrics. Si usa el [escalado horizontal](analysis-services-scale-out.md) de consultas y necesita separar las métricas de cada réplica de lectura, use la tabla AzureDiagnostics en su lugar, donde **OperationName** (Nombre de operación) es igual a **LogMetric**.

## <a name="setup-diagnostics-logging"></a>Configuración del registro de diagnósticos

### <a name="azure-portal"></a>Azure portal

1. En [Azure Portal](https://portal.azure.com) > servidor, haga clic en **Configuración de diagnóstico** en el panel de navegación izquierdo y luego en **Activar diagnóstico**.

    ![Activación del registro de recursos para Azure Cosmos DB en Azure Portal](./media/analysis-services-logging/aas-logging-turn-on-diagnostics.png)

2. En **Configuración de diagnóstico**, especifique las opciones siguientes: 

    * **Nombre**. Escriba un nombre para los registros que quiere crear.

    * **Archivar en una cuenta de almacenamiento**. Para usar esta opción, necesita una cuenta de almacenamiento existente a la cual conectarse. Vea [Crear una cuenta de almacenamiento](../storage/common/storage-account-create.md). Siga las instrucciones para crear un administrador de recursos, una cuenta de propósito general y, a continuación, seleccione la cuenta de almacenamiento volviendo a esta página en el portal. Las cuentas de almacenamiento recién creadas pueden tardar unos minutos en aparecer en el menú desplegable.
    * **Transmitir a un centro de eventos**. Para usar esta opción, necesita un espacio de nombres de Event Hubs y un centro de eventos al que conectarse. Para más información, consulte [Creación de un espacio de nombres de Event Hubs y un centro de eventos con Azure Portal](../event-hubs/event-hubs-create.md). Luego, vuelva a esta página en el portal para seleccionar el espacio de nombres de Event Hubs y el nombre de la directiva.
    * **Enviar a Azure Monitor (área de trabajo de Log Analytics)** . Para usar esta opción, puede utilizar un área de trabajo ya existente o [crear un nuevo recurso de área de trabajo](../azure-monitor/logs/quick-create-workspace.md) en el portal. Para más información acerca de cómo ver los registros de Log Analytics, consulte [Visualización de los registros en el área de trabajo de Log Analytics](#view-logs-in-log-analytics-workspace) en este artículo.

    * **Motor**. Seleccione esta opción para registrar xEvents. Si va a archivar en una cuenta de almacenamiento, puede seleccionar el período de retención para los registros de recursos. Los registros se eliminan automáticamente una vez expira el período de retención.
    * **Servicio**. Seleccione esta opción para registrar los eventos del nivel de servicio. Si va a archivar en una cuenta de almacenamiento, puede seleccionar el período de retención para los registros de recursos. Los registros se eliminan automáticamente una vez expira el período de retención.
    * **Métricas**. Seleccione esta opción para almacenar los datos detallados en [Métricas](analysis-services-monitor.md#server-metrics). Si va a archivar en una cuenta de almacenamiento, puede seleccionar el período de retención para los registros de recursos. Los registros se eliminan automáticamente una vez expira el período de retención.

3. Haga clic en **Save**(Guardar).

    Si recibe un error que indica que "No se pudo actualizar el diagnóstico de \<workspace name>. La suscripción \<subscription id> no está registrada para usar microsoft.insights", siga las instrucciones para [solucionar problemas de Azure Diagnostics](../azure-monitor/essentials/resource-logs.md) a fin de registrar la cuenta; a continuación, vuelva a intentar este procedimiento.

    Si desea cambiar el modo en que se guardan los registros de recursos en algún momento, puede volver a esta página para modificar la configuración.

### <a name="powershell"></a>PowerShell

Estos son los comandos básicos para comenzar. Si desea obtener ayuda paso a paso para configurar el registro en una cuenta de almacenamiento con PowerShell, vea el tutorial más adelante en este artículo.

Para habilitar las métricas y los registros de recursos con PowerShell, use los siguientes comandos:

- Para habilitar el almacenamiento de registros de recursos en una cuenta de almacenamiento, use este comando:

   ```powershell
   Set-AzDiagnosticSetting -ResourceId [your resource id] -StorageAccountId [your storage account id] -Enabled $true
   ```

   El identificador de la cuenta de almacenamiento es el identificador de recurso para la cuenta de almacenamiento a la que desea enviar los registros.

- Para habilitar el streaming de registros de recursos en un centro de eventos, use este comando:

   ```powershell
   Set-AzDiagnosticSetting -ResourceId [your resource id] -ServiceBusRuleId [your service bus rule id] -Enabled $true
   ```

   El identificador de regla de Azure Service Bus es una cadena con este formato:

   ```powershell
   {service bus resource ID}/authorizationrules/{key name}
   ``` 

- Para habilitar el envío de registros de recursos a un área de trabajo de Log Analytics, use este comando:

   ```powershell
   Set-AzDiagnosticSetting -ResourceId [your resource id] -WorkspaceId [resource id of the log analytics workspace] -Enabled $true
   ```

- Puede obtener el identificador de recurso de su área de trabajo de Log Analytics con el comando siguiente:

   ```powershell
   (Get-AzOperationalInsightsWorkspace).ResourceId
   ```

Puede combinar estos parámetros para habilitar varias opciones de salida.

### <a name="rest-api"></a>API DE REST

Obtenga información sobre cómo [cambiar la configuración de diagnóstico mediante la API de REST de Azure Monitor](/rest/api/monitor/). 

### <a name="resource-manager-template"></a>Plantilla de Resource Manager

Lea sobre cómo [habilitar la configuración de diagnóstico al crear recursos con la plantilla de Resource Manager](../azure-monitor/essentials/resource-manager-diagnostic-settings.md). 

## <a name="manage-your-logs"></a>Administración de los registros

Los registros normalmente estarán disponibles en un par de horas tras la configuración del registro. Es decisión suya administrar los registros en la cuenta de almacenamiento:

* Use los métodos de control de acceso estándar de Azure para proteger los registros mediante la restricción de los usuarios que pueden entrar a ellos.
* Elimine los registros que ya no desee mantener en la cuenta de almacenamiento.
* Asegúrese de establecer un período de retención porque los registros antiguos se eliminan de la cuenta de almacenamiento.

## <a name="view-logs-in-log-analytics-workspace"></a>Visualización de los registros en el área de trabajo de Log Analytics

Los eventos del servidor y las métricas se integran con xEvents en el recurso de área de trabajo de Log Analytics para realizar un análisis en paralelo. El área de trabajo de Log Analytics también puede configurarse para recibir eventos de otros servicios de Azure proporcionando una vista holística de los datos de registro de diagnóstico en la arquitectura.

Para ver los datos de diagnóstico, en el área de trabajo de Log Analytics, abra **Registros** en el menú izquierdo.

![Opciones de búsqueda de registros en Azure Portal](./media/analysis-services-logging/aas-logging-open-log-search.png)

En el generador de consultas, expanda **LogManagement** > **AzureDiagnostics**. AzureDiagnostics incluye eventos de motor y de servicio. Observe que se crea una consulta sobre la marcha. El campo EventClass\_s contiene los nombres de xEvent, lo que puede resultar familiar si se ha usado xEvents para el registro local. Haga clic en **EventClass\_s** o en uno de los nombres de evento, el área de trabajo de Log Analytics continúa generando una consulta. Asegúrese de guardar las consultas para reutilizarlas más adelante.

### <a name="example-queries"></a>Consultas de ejemplo

#### <a name="example-1"></a>Ejemplo 1

La consulta siguiente devuelve las duraciones de cada evento de fin o actualización de consulta para una base de datos modelo y el servidor: Si se escala horizontalmente, los resultados se desglosan por réplica porque el número de réplica está incluido en ServerName_s. Al agrupar por RootActivityId_g, se reduce el número de filas recuperadas de la API REST de Azure Diagnostics y se contribuye a mantenerlas dentro de los límites, tal y como se describe en [Límites de velocidad de Log Analytics](https://dev.loganalytics.io/documentation/Using-the-API/Limits).

```Kusto
let window = AzureDiagnostics
   | where ResourceProvider == "MICROSOFT.ANALYSISSERVICES" and Resource =~ "MyServerName" and DatabaseName_s =~ "MyDatabaseName" ;
window
| where OperationName has "QueryEnd" or (OperationName has "CommandEnd" and EventSubclass_s == 38)
| where extract(@"([^,]*)", 1,Duration_s, typeof(long)) > 0
| extend DurationMs=extract(@"([^,]*)", 1,Duration_s, typeof(long))
| project  StartTime_t,EndTime_t,ServerName_s,OperationName,RootActivityId_g,TextData_s,DatabaseName_s,ApplicationName_s,Duration_s,EffectiveUsername_s,User_s,EventSubclass_s,DurationMs
| order by StartTime_t asc
```

#### <a name="example-2"></a>Ejemplo 2

La siguiente consulta devuelve el consumo de memoria y QPU para un servidor. Si se escala horizontalmente, los resultados se desglosan por réplica porque el número de réplica está incluido en ServerName_s.

```Kusto
let window = AzureDiagnostics
   | where ResourceProvider == "MICROSOFT.ANALYSISSERVICES" and Resource =~ "MyServerName";
window
| where OperationName == "LogMetric" 
| where name_s == "memory_metric" or name_s == "qpu_metric"
| project ServerName_s, TimeGenerated, name_s, value_s
| summarize avg(todecimal(value_s)) by ServerName_s, name_s, bin(TimeGenerated, 1m)
| order by TimeGenerated asc 
```

#### <a name="example-3"></a>Ejemplo 3

La siguiente consulta devuelve los contadores de rendimiento de filas leídas/s del motor de Analysis Services para un servidor.

```Kusto
let window =  AzureDiagnostics
   | where ResourceProvider == "MICROSOFT.ANALYSISSERVICES" and Resource =~ "MyServerName";
window
| where OperationName == "LogMetric" 
| where parse_json(tostring(parse_json(perfobject_s).counters))[0].name == "Rows read/sec" 
| extend Value = tostring(parse_json(tostring(parse_json(perfobject_s).counters))[0].value) 
| project ServerName_s, TimeGenerated, Value
| summarize avg(todecimal(Value)) by ServerName_s, bin(TimeGenerated, 1m)
| order by TimeGenerated asc 
```

Existen cientos de consultas que puede utilizar. Para más información acerca de las consultas, vea [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/logs/get-started-queries.md).


## <a name="turn-on-logging-by-using-powershell"></a>Activación del registro con PowerShell

En este tutorial rápido, se crea una cuenta de almacenamiento en la misma suscripción y grupo de recursos que el servidor de Analysis Services. A continuación, se usa Set-AzDiagnosticSetting para activar el registro de diagnóstico y se envía la salida a la nueva cuenta de almacenamiento.

### <a name="prerequisites"></a>Requisitos previos
Para realizar este tutorial, necesitará los recursos siguientes:

* Un servidor de Azure Analysis Services. Para obtener instrucciones sobre cómo crear un recurso de servidor, consulte [Creación de un servidor en Azure Portal](analysis-services-create-server.md) o [Creación de un servidor de Azure Analysis Services mediante PowerShell](analysis-services-create-powershell.md).

### <a name="aconnect-to-your-subscriptions"></a></a>Conexión a las suscripciones

Inicie una sesión de PowerShell de Azure e inicie sesión en su cuenta de Azure con el siguiente comando:  

```powershell
Connect-AzAccount
```

En la ventana emergente del explorador, escriba el nombre de usuario y la contraseña de su cuenta de Azure. Azure PowerShell obtiene todas las suscripciones asociadas a esta cuenta y, de forma predeterminada, usa la primera.

Si tiene varias suscripciones, es posible que deba especificar la que se usó para crear su instancia de Azure Key Vault. Escriba lo siguiente para ver las suscripciones de su cuenta:

```powershell
Get-AzSubscription
```

A continuación, para especificar la suscripción asociada a la cuenta de Azure Analysis Services que registrará, escriba:

```powershell
Set-AzContext -SubscriptionId <subscription ID>
```

> [!NOTE]
> Si tiene varias suscripciones asociadas a su cuenta, es importante especificar la suscripción.
>
>

### <a name="create-a-new-storage-account-for-your-logs"></a>Creación de una cuenta de almacenamiento nueva para los registros

Puede usar una cuenta de almacenamiento existente para los registros, siempre que sea en la misma suscripción que el servidor. Para este tutorial, creará una nueva cuenta de almacenamiento dedicada a los registros de Analysis Services. Para facilitar el proceso, almacenamos los detalles de la cuenta de almacenamiento en una variable denominada **sa**.

También puede utilizar el mismo grupo de recursos que la carpeta que contenga el servidor de Analysis Services. Reemplace los valores de `awsales_resgroup`, `awsaleslogs` y `West Central US` con los suyos:

```powershell
$sa = New-AzStorageAccount -ResourceGroupName awsales_resgroup `
-Name awsaleslogs -Type Standard_LRS -Location 'West Central US'
```

### <a name="identify-the-server-account-for-your-logs"></a>Identificación de la cuenta del servidor para los registros

Establezca el nombre de la cuenta en una variable denominada **account**, donde ResourceName es el nombre de la cuenta.

```powershell
$account = Get-AzResource -ResourceGroupName awsales_resgroup `
-ResourceName awsales -ResourceType "Microsoft.AnalysisServices/servers"
```

### <a name="enable-logging"></a>Habilitar registro

Para habilitar el registro, use el cmdlet Set-AzDiagnosticSetting, junto con las variables de la nueva cuenta de almacenamiento, la cuenta de servidor y la categoría. Ejecute el siguiente comando y establezca la marca **-Enabled** en **$true**:

```powershell
Set-AzDiagnosticSetting  -ResourceId $account.ResourceId -StorageAccountId $sa.Id -Enabled $true -Categories Engine
```

El resultado debe parecerse a este ejemplo:

```powershell
StorageAccountId            : 
/subscriptions/a23279b5-xxxx-xxxx-xxxx-47b7c6d423ea/resourceGroups/awsales_resgroup/providers/Microsoft.Storage/storageAccounts/awsaleslogs
ServiceBusRuleId            :
EventHubAuthorizationRuleId :
Metrics                    
    TimeGrain       : PT1M
    Enabled         : False
    RetentionPolicy
    Enabled : False
    Days    : 0


Logs                       
    Category        : Engine
    Enabled         : True
    RetentionPolicy
    Enabled : False
    Days    : 0


    Category        : Service
    Enabled         : False
    RetentionPolicy
    Enabled : False
    Days    : 0


WorkspaceId                 :
Id                          : /subscriptions/a23279b5-xxxx-xxxx-xxxx-47b7c6d423ea/resourcegroups/awsales_resgroup/providers/microsoft.analysisservic
es/servers/awsales/providers/microsoft.insights/diagnosticSettings/service
Name                        : service
Type                        :
Location                    :
Tags                        :
```

Este resultado confirma que el registro está habilitado ahora para el servidor, y que la información se guarda en la cuenta de almacenamiento.

También puede establecer una directiva de retención para los registros, de forma que los más antiguos se eliminen automáticamente. Por ejemplo, establezca la directiva de retención estableciendo la marca **-RetentionEnabled** en **$true** y el parámetro **-RetentionInDays** en **90**. Los registros de más de 90 días se eliminan automáticamente.

```powershell
Set-AzDiagnosticSetting -ResourceId $account.ResourceId`
 -StorageAccountId $sa.Id -Enabled $true -Categories Engine`
  -RetentionEnabled $true -RetentionInDays 90
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el [registro de recursos en Azure Monitor](../azure-monitor/essentials/platform-logs-overview.md).

Consulte [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) en la Ayuda de PowerShell.
