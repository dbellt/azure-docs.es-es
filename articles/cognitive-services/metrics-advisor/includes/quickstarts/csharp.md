---
title: Inicio rápido de C# para Metrics Advisor
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: applied-ai-services
ms.subservice: metrics-advisor
ms.topic: include
ms.date: 11/09/2020
ms.author: mbullwin
ms.openlocfilehash: 190982f29bdae30f6937ba648cc8de42eb147623
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111894479"
---
[Documentación de referencia](/dotnet/api/overview/azure/ai.metricsadvisor-readme-pre) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/metricsadvisor/Azure.AI.MetricsAdvisor/src) | [Paquete (NuGet)](https://www.nuget.org/packages/Azure.AI.MetricsAdvisor) | [Ejemplos](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/metricsadvisor/Azure.AI.MetricsAdvisor/samples/README.md)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* La versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Una vez que tenga la suscripción de Azure, <a href="https://go.microsoft.com/fwlink/?linkid=2142156"  title="cree un recurso de Metrics Advisor"  target="_blank">create a Metrics Advisor resource </a> en Azure Portal para implementar la instancia de Metrics Advisor.  
* Su propia base de datos SQL con datos de series temporales.
   
> [!TIP]
> * Puede encontrar ejemplos de Metrics Advisor de .NET en [GitHub](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/metricsadvisor/Azure.AI.MetricsAdvisor/samples/README.md).
> * El recurso de Metrics Advisor puede tardar entre 10 y 30 minutos en implementar una instancia de servicio para que pueda usarla. Haga clic en **Ir al recurso** una vez que se implemente correctamente. Después de la implementación, puede empezar a usar la instancia de Metrics Advisor con el portal web y la API de REST. 
> * Puede encontrar la dirección URL de la API de REST en Azure Portal, en la sección **Información general** de su recurso. Tendrá este aspecto:
>    * `https://<instance-name>.cognitiveservices.azure.com/`
   
## <a name="setting-up"></a>Instalación

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente 

Después de crear un proyecto, instale la biblioteca cliente; para ello, haga clic con el botón derecho en la solución del proyecto en el **Explorador de soluciones** y seleccione **Administrar paquetes NuGet**. En el administrador de paquetes que se abre, seleccione **Examinar** **Incluir versión preliminar** y busque `Azure.AI.MetricsAdvisor`. Seleccione la versión `1.0.0-beta.2` e **Instalar**. 

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `metrics-advisor-quickstart`. Este comando crea un sencillo proyecto de C#, "Hola mundo", con un solo archivo de origen: *program.cs*. 

```console
dotnet new console -n metrics-advisor-quickstart
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```console
dotnet build
```

La salida de la compilación no debe contener advertencias ni errores. 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente 

Si usa un IDE que no sea Visual Studio, puede instalar la biblioteca cliente de Metrics Advisor para .NET con el siguiente comando:

```console
dotnet add package Azure.AI.MetricsAdvisor --version 1.0.0-beta.2
```

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/Azure.AI.MetricsAdvisor_1.0.0-beta.1/sdk/metricsadvisor/Azure.AI.MetricsAdvisor/samples), que contiene los ejemplos de código de este inicio rápido.

En el directorio del proyecto, abra el archivo *program.cs* y agregue lo siguiente mediante directivas `using`:

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Azure.AI.MetricsAdvisor.Administration;
using Azure.AI.MetricsAdvisor;
using Azure.AI.MetricsAdvisor.Models;
```

En el método `Main()` de la aplicación, agregue llamadas para los métodos que se usan en este inicio rápido. Las creará más adelante.

```csharp
static void Main(string[] args){
    // You will create the below methods later in the quickstart
    exampleTask1();
}
```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases controlan algunas de las características principales del SDK de C# para Metrics Advisor.

|Nombre|Descripción|
|---|---|
| [MetricsAdvisorClient](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/metricsadvisor/Azure.AI.MetricsAdvisor/src/MetricsAdvisorClient.cs) | **Se usa para:** <br> - Enumerar pendientes <br> - Enumerar la causa raíz de los incidentes <br> - Recuperar los datos de series temporales originales y los datos de series temporales enriquecidos por el servicio. <br> - Enumerar alertas <br> - Agregar comentarios para optimizar el modelo |
| [MetricsAdvisorAdministrationClient](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/metricsadvisor/Azure.AI.MetricsAdvisor/src/MetricsAdvisorAdministrationClient.cs)| **Permite:** <br> - Administrar fuentes de distribución de datos <br> - Configurar la detección de anomalías <br> - Configurar las alertas de anomalías <br> - Administrar enlaces  |
| [DataFeed](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/metricsadvisor/Azure.AI.MetricsAdvisor/src/Models/DataFeed/DataFeed.cs)| **Lo que Metrics Advisor ingiere de su origen de datos. `DataFeed` contiene filas de:** <br> - Marcas de tiempo <br> - Cero o más dimensiones <br> - Una o varias medidas  |
| [DataFeedMetric](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/metricsadvisor/Azure.AI.MetricsAdvisor/src/Models/DataFeed/DataFeedMetric.cs)| `DataFeedMetric` es una medida cuantificable que se usa para supervisar y evaluar el estado de un proceso empresarial específico. Puede ser una combinación de varios valores de serie temporal divididos en dimensiones. Por ejemplo, una métrica de mantenimiento de la Web podría contener dimensiones para recuento de usuarios y el mercado es-es. |

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes tareas con la biblioteca cliente de Metrics Advisor para .NET:

* [Autenticar el cliente](#authenticate-the-client)
* [Adición de una fuente de distribución de datos](#add-a-data-feed)
* [Comprobación del estado de la ingesta](#check-the-ingestion-status)
* [Configuración de la detección de anomalías](#configure-anomaly-detection)
* [Crear un enlace](#create-a-hook)
* [Crear una configuración de alertas](#create-an-alert-configuration)
* [Consultar la alerta](#query-the-alert)

## <a name="authenticate-the-client"></a>Autenticar el cliente

En la clase `Program` de la aplicación, cree variables para las claves y el punto de conexión del recurso.

> [!IMPORTANT]
> Vaya a Azure Portal. Si el recurso de Metrics Advisor que creó en la sección **Requisitos previos** se implementó correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Sus claves de suscripción y su punto de conexión se encuentran en la página **Key and Endpoint** (Clave y punto de conexión) del recurso, en **Administración de recursos**. <br><br>Para recuperar la clave de API, debe ir a [https://metricsadvisor.azurewebsites.net](https://metricsadvisor.azurewebsites.net). Seleccione los valores apropiados: **Directory** (Directorio), **Subscriptions** (Suscripciones) y **Workspace** (Área de trabajo) del recurso y seleccione **Comenzar**. A continuación, podrá recuperar las claves de API de [https://metricsadvisor.azurewebsites.net/api-key](https://metricsadvisor.azurewebsites.net/api-key).   
>
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Para más información, consulte el artículo sobre la [seguridad](../../../cognitive-services-security.md) de Cognitive Services.

Una vez que tenga la suscripción y las claves de API, cree MetricsAdvisorKeyCredential. Con el punto de conexión y la credencial de la clave puede crear [`MetricsAdvisorClient`](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/metricsadvisor/Azure.AI.MetricsAdvisor/src/MetricsAdvisorClient.cs):

```csharp
string endpoint = "<endpoint>";
string subscriptionKey = "<subscriptionKey>";
string apiKey = "<apiKey>";
var credential = new MetricsAdvisorKeyCredential(subscriptionKey, apiKey);
var client = new MetricsAdvisorClient(new Uri(endpoint), credential);
```

También puede crear una instancia de [`MetricsAdvisorAdministrationClient`](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/metricsadvisor/Azure.AI.MetricsAdvisor/src/MetricsAdvisorAdministrationClient.cs) para realizar operaciones administrativas:

```csharp
string endpoint = "<endpoint>";
string subscriptionKey = "<subscriptionKey>";
string apiKey = "<apiKey>";
var credential = new MetricsAdvisorKeyCredential(subscriptionKey, apiKey);
var adminClient = new MetricsAdvisorAdministrationClient(new Uri(endpoint), credential);
```

## <a name="add-a-data-feed"></a>Adición de una fuente de distribución de datos

Metrics Advisor admite varios tipos de orígenes de datos. En este ejemplo se ilustrará una instancia de `DataFeed` que extrae datos de un servidor de SQL Server. Reemplace `connection_String` por su propia cadena de conexión de SQL Server y reemplace `query` por una consulta que devuelva los datos en una sola marca de tiempo. También tendrá que ajustar los valores de `DataFeedMetric` y `DataFeedDimension` en función de los datos personalizados.


```csharp
string sqlServerConnectionString = "<connection_String>";
string sqlServerQuery = "<query>";

var dataFeedName = "Sample data feed";
var dataFeedSource = new SqlServerDataFeedSource(sqlServerConnectionString, sqlServerQuery);
var dataFeedGranularity = new DataFeedGranularity(DataFeedGranularityType.Daily);

var dataFeedMetrics = new List<DataFeedMetric>()
{
    new DataFeedMetric("cost"),
    new DataFeedMetric("revenue")
};
var dataFeedDimensions = new List<DataFeedDimension>()
{
    new DataFeedDimension("category"),
    new DataFeedDimension("city")
};
var dataFeedSchema = new DataFeedSchema(dataFeedMetrics)
{
    DimensionColumns = dataFeedDimensions
};

var ingestionStartTime = DateTimeOffset.Parse("2020-01-01T00:00:00Z");
var dataFeedIngestionSettings = new DataFeedIngestionSettings(ingestionStartTime);

var dataFeed = new DataFeed(dataFeedName, dataFeedSource, dataFeedGranularity, dataFeedSchema, dataFeedIngestionSettings);

Response<string> response = await adminClient.CreateDataFeedAsync(dataFeed);

string dataFeedId = response.Value;

Console.WriteLine($"Data feed ID: {dataFeedId}");

// Note that only the ID of the data feed is known at this point. You can perform another
// service call to GetDataFeedAsync or GetDataFeed to get more information, such as status,
// created time, the list of administrators, or the metric IDs.

Response<DataFeed> response = await adminClient.GetDataFeedAsync(dataFeedId);

DataFeed dataFeed = response.Value;

Console.WriteLine($"Data feed status: {dataFeed.Status.Value}");
Console.WriteLine($"Data feed created time: {dataFeed.CreatedTime.Value}");

Console.WriteLine($"Data feed administrators:");
foreach (string admin in dataFeed.Administrators)
{
    Console.WriteLine($" - {admin}");
}

Console.WriteLine($"Metric IDs:");
foreach (DataFeedMetric metric in dataFeed.Schema.MetricColumns)
{
    Console.WriteLine($" - {metric.MetricName}: {metric.MetricId}");
}
```

## <a name="check-the-ingestion-status"></a>Comprobación del estado de la ingesta

Compruebe el estado de la ingesta de una instancia de `DataFeed` creada anteriormente

```csharp
string dataFeedId = "<dataFeedId>";

var startTime = DateTimeOffset.Parse("2020-01-01T00:00:00Z");
var endTime = DateTimeOffset.Parse("2020-09-09T00:00:00Z");
var options = new GetDataFeedIngestionStatusesOptions(startTime, endTime)
{
    TopCount = 5
};

Console.WriteLine("Ingestion statuses:");
Console.WriteLine();

int statusCount = 0;

await foreach (DataFeedIngestionStatus ingestionStatus in adminClient.GetDataFeedIngestionStatusesAsync(dataFeedId, options))
{
    Console.WriteLine($"Timestamp: {ingestionStatus.Timestamp}");
    Console.WriteLine($"Status: {ingestionStatus.Status}");
    Console.WriteLine($"Service message: {ingestionStatus.Message}");
    Console.WriteLine();

    // Print at most 5 statuses.
    if (++statusCount >= 5)
    {
        break;
    }
}
```

## <a name="configure-anomaly-detection"></a>Configuración de la detección de anomalías 

Cree una configuración de detección de anomalías para indicar al servicio qué puntos de datos deberían considerarse anomalías.

```csharp
string metricId = "<metricId>";
string configurationName = "Sample anomaly detection configuration";

var hardThresholdSuppressCondition = new SuppressCondition(1, 100);
var hardThresholdCondition = new HardThresholdCondition(AnomalyDetectorDirection.Down, hardThresholdSuppressCondition)
{
    LowerBound = 5.0
};

var smartDetectionSuppressCondition = new SuppressCondition(4, 50);
var smartDetectionCondition = new SmartDetectionCondition(10.0, AnomalyDetectorDirection.Up, smartDetectionSuppressCondition);

var detectionCondition = new MetricWholeSeriesDetectionCondition()
{
    HardThresholdCondition = hardThresholdCondition,
    SmartDetectionCondition = smartDetectionCondition,
    CrossConditionsOperator = DetectionConditionsOperator.Or
};

var detectionConfiguration = new AnomalyDetectionConfiguration(metricId, configurationName, detectionCondition);

Response<string> response = await adminClient.CreateDetectionConfigurationAsync(detectionConfiguration);

string detectionConfigurationId = response.Value;

Console.WriteLine($"Anomaly detection configuration ID: {detectionConfigurationId}");
```

### <a name="create-a-hook"></a>Creación de un enlace

Metrics Advisor admite las clases `EmailNotificationHook` y `WebNotificationHook` como medio de suscribirse a notificaciones de alertas. En este ejemplo ilustraremos cómo crear `EmailNotificationHook`. Para empezar a obtener notificaciones, es preciso que pase el enlace a una configuración de aleta de anomalía. Para más información, consulte la [creación de una configuración de alerta de anomalías](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/metricsadvisor/Azure.AI.MetricsAdvisor#create-an-anomaly-alert-configuration) de ejemplo.

```csharp
string hookName = "Sample hook";
var emailsToAlert = new List<string>()
{
    "email1@sample.com",
    "email2@sample.com"
};

var emailHook = new EmailNotificationHook(hookName, emailsToAlert);

Response<string> response = await adminClient.CreateHookAsync(emailHook);

string hookId = response.Value;

Console.WriteLine($"Hook ID: {hookId}");
```

##  <a name="create-an-alert-configuration"></a>Creación de una configuración de alertas

Cree `AnomalyAlertConfiguration` para indicar al servicio qué anomalías deben desencadenar alertas.

```csharp
string hookId = "<hookId>";
string anomalyDetectionConfigurationId = "<anomalyDetectionConfigurationId>";

string configurationName = "Sample anomaly alert configuration";
var idsOfHooksToAlert = new List<string>() { hookId };

var scope = MetricAnomalyAlertScope.GetScopeForWholeSeries();
var metricAlertConfigurations = new List<MetricAnomalyAlertConfiguration>()
{
    new MetricAnomalyAlertConfiguration(anomalyDetectionConfigurationId, scope)
};

AnomalyAlertConfiguration alertConfiguration = new AnomalyAlertConfiguration(configurationName, idsOfHooksToAlert, metricAlertConfigurations);

Response<string> response = await adminClient.CreateAlertConfigurationAsync(alertConfiguration);

string alertConfigurationId = response.Value;

Console.WriteLine($"Alert configuration ID: {alertConfigurationId}");
```

### <a name="query-the-alert"></a>Consulta de la alerta

Examine las alertas creadas por una determinada configuración de alertas de anomalías.

```csharp
string anomalyAlertConfigurationId = "<anomalyAlertConfigurationId>";

var startTime = DateTimeOffset.Parse("2020-01-01T00:00:00Z");
var endTime = DateTimeOffset.UtcNow;
var options = new GetAlertsOptions(startTime, endTime, AlertQueryTimeMode.AnomalyTime)
{
    TopCount = 5
};

int alertCount = 0;

await foreach (AnomalyAlert alert in client.GetAlertsAsync(anomalyAlertConfigurationId, options))
{
    Console.WriteLine($"Alert created at: {alert.CreatedTime}");
    Console.WriteLine($"Alert at timestamp: {alert.Timestamp}");
    Console.WriteLine($"Id: {alert.Id}");
    Console.WriteLine();

    // Print at most 5 alerts.
    if (++alertCount >= 5)
    {
        break;
    }
}
```

Una vez que conozca el identificador de una alerta, enumere las [anomalías](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/metricsadvisor/Azure.AI.MetricsAdvisor/README.md#data-point-anomaly) que ha desencadenado dicha alerta.

```csharp
string alertConfigurationId = "<alertConfigurationId>";
string alertId = "<alertId>";

var options = new GetAnomaliesForAlertOptions() { TopCount = 3 };

int anomalyCount = 0;

await foreach (DataPointAnomaly anomaly in client.GetAnomaliesAsync(alertConfigurationId, alertId, options))
{
    Console.WriteLine($"Anomaly detection configuration ID: {anomaly.AnomalyDetectionConfigurationId}");
    Console.WriteLine($"Metric ID: {anomaly.MetricId}");
    Console.WriteLine($"Anomaly at timestamp: {anomaly.Timestamp}");
    Console.WriteLine($"Anomaly detected at: {anomaly.CreatedTime}");
    Console.WriteLine($"Status: {anomaly.Status}");
    Console.WriteLine($"Severity: {anomaly.Severity}");
    Console.WriteLine("Series key:");

    foreach (KeyValuePair<string, string> keyValuePair in anomaly.SeriesKey.AsDictionary())
    {
        Console.WriteLine($"  Dimension '{keyValuePair.Key}': {keyValuePair.Value}");
    }

    Console.WriteLine();

    // Print at most 3 anomalies.
    if (++anomalyCount >= 3)
    {
        break;
    }
}
```


### <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```dotnet
dotnet run
```