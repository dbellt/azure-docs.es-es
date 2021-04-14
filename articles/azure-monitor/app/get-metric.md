---
title: Obtención de métricas en Application Insights de Azure Monitor
description: Aprenda a usar eficazmente la llamada a GetMetric() para capturar métricas previamente agregadas de forma local para aplicaciones .NET y .NET Core con Application Insights de Azure Monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 04/28/2020
ms.openlocfilehash: 22baa1ae9554601a72ffdb848b87d99281067967
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106384296"
---
# <a name="custom-metric-collection-in-net-and-net-core"></a>Recopilación de métricas personalizadas en .NET y .NET Core

Los SDK de Application Insights de Azure Monitor para .NET y .NET Core tienen dos métodos diferentes para recopilar métricas personalizadas, `TrackMetric()`y `GetMetric()`. La diferencia principal entre estos dos métodos es la agregación local. `TrackMetric()` no dispone de agregación previa, en tanto que `GetMetric()` tiene agregación previa. El enfoque recomendado es usar agregación, por lo que `TrackMetric()` ya no es el método preferido para recopilar métricas personalizadas. En este artículo se le guiará por el uso del método GetMetric () y parte de la lógica en la que se basa en su funcionamiento.

## <a name="pre-aggregating-vs-non-pre-aggregating-api"></a>API de agregación previa frente a la de no agregación previa

`TrackMetric()` envía datos de telemetría sin procesar que denotan una métrica. No es eficaz para enviar un elemento de telemetría único para cada valor. `TrackMetric()` también es ineficaz en términos de rendimiento, ya que cada elemento `TrackMetric(item)` pasa a través de la canalización completa del SDK de inicializadores y procesadores de telemetría. A diferencia de `TrackMetric()`, `GetMetric()` controla la agregación previa local automáticamente y, a continuación, solo envía una métrica de resumen agregada cada intervalo fijo de un minuto. Por lo tanto, si necesita supervisar de cerca algunas métricas personalizadas en el nivel de segundos o incluso en el de milisegundos, puede hacerlo y además solo se incurre en el costo de almacenamiento y tráfico de red de la supervisión en cada minuto. Esto también reduce en gran medida el riesgo de que se produzca una limitación, ya que se reduce considerablemente el número total de elementos de telemetría que se deben enviar para una métrica agregada.

En Application Insights, las métricas personalizadas recopiladas mediante `TrackMetric()` y `GetMetric()` no están sujetas a [muestreo](./sampling.md). El muestreo de métricas importantes puede dar lugar a escenarios en los que las alertas que se puedan haber creado en torno a esas métricas resulten poco confiables. Si nunca se realiza el muestreo de las métricas personalizadas, normalmente puede estar seguro de que se activará una alerta cuando se infrinjan los umbrales de alerta.  Sin embargo, dado que no se muestrean las métricas personalizadas, existen algunas posibles preocupaciones.

Si necesita realizar un seguimiento de las tendencias de una métrica cada segundo o en un intervalo aún más granular, esto puede dar lugar a:

- Aumento de los costos de almacenamiento de datos. Hay un costo asociado con la cantidad de datos que se envían a Azure Monitor. (Cuantos más datos envíe, mayor será el costo total de la supervisión).
- Aumento del tráfico de red y sobrecarga del rendimiento. (En algunos escenarios esto podría tener un costo monetario y de rendimiento de la aplicación).
- Riesgo de limitación de la ingesta de datos. (El servicio Azure Monitor descarta ("limita") puntos de datos cuando la aplicación envía un número muy elevado de datos de telemetría en un intervalo de tiempo corto).

La limitación tiene una preocupación especial en la que, al igual que en el muestreo, la limitación puede conducir a alertas perdidas, ya que la condición para desencadenar una alerta podría producirse localmente y, a continuación, ser eliminada en el punto de conexión de ingesta debido a que se están enviando demasiados datos. Por este motivo, para .NET y .NET Core, no se recomienda usar `TrackMetric()` a menos que haya implementado su propia lógica de agregación local. Si intenta realizar un seguimiento de cada instancia en la que se produce un evento durante un período de tiempo determinado, es posible que sea mejor opción [`TrackEvent()`](./api-custom-events-metrics.md#trackevent). Sin embargo, tenga en cuenta que, a diferencia de las métricas personalizadas, los eventos personalizados están sujetos a muestreo. Por supuesto, puede utilizar `TrackMetric()` incluso sin escribir su propia agregación previa local, pero si lo hace, tenga en cuenta los posibles problemas.

En resumen, `GetMetric()` es el enfoque recomendado, ya que realiza la agregación previa, acumula los valores de todas las llamadas a Track() y envía un resumen o agregado cada minuto. Esto puede reducir significativamente los costos y la sobrecarga de rendimiento, ya que envía menos puntos de datos, pero recopila toda la información pertinente.

> [!NOTE]
> Solo los SDK de .NET y .NET Core tienen el método GetMetric(). Si usa Java, puede usar las [métricas de Micrometer](./micrometer-java.md) o `TrackMetric()`. Para JavaScript y Node.js puede utilizar `TrackMetric()`, pero tenga en cuenta las advertencias que se han descrito en la sección anterior. En el caso de Python, puede usar [OpenCensus.stats](./opencensus-python.md#metrics) para enviar métricas personalizadas, pero la implementación de las métricas es diferente.

## <a name="getting-started-with-getmetric"></a>Introducción a GetMetric

En los ejemplos, vamos a usar una aplicación básica de servicio de trabajo de .NET Core 3.1. Si desea replicar exactamente el entorno de prueba utilizado con estos ejemplos, siga los pasos 1 a 6 del artículo sobre [supervisión del servicio de trabajo](./worker-service.md#net-core-30-worker-service-application) para agregar Application Insights a una plantilla de proyecto básica de servicio de trabajo. Estos conceptos se aplican a cualquier aplicación general en la que se pueda usar el SDK, incluidas las aplicaciones web y las aplicaciones de consola.

### <a name="sending-metrics"></a>Envío de métricas

Reemplace el contenido del archivo `worker.cs` por lo siguiente:

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using Microsoft.ApplicationInsights;

namespace WorkerService3
{
    public class Worker : BackgroundService
    {
        private readonly ILogger<Worker> _logger;
        private TelemetryClient _telemetryClient;

        public Worker(ILogger<Worker> logger, TelemetryClient tc)
        {
            _logger = logger;
            _telemetryClient = tc;
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {   // The following line demonstrates usages of GetMetric API.
            // Here "computersSold", a custom metric name, is being tracked with a value of 42 every second.
            while (!stoppingToken.IsCancellationRequested)
            {
                _telemetryClient.GetMetric("ComputersSold").TrackValue(42);

                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
                await Task.Delay(1000, stoppingToken);
            }
        }
    }
}
```

Si ejecuta el código anterior y observa la telemetría que se envía mediante la ventana de salida de Visual Studio o una herramienta como Fiddler de Telerik, verá que el bucle while se ejecuta repetidamente sin que se envíen datos de telemetría y, a continuación, se envía un único elemento de telemetría en torno a la marca de 60 segundos, que en el caso de nuestra prueba tiene el siguiente aspecto:

```json
Application Insights Telemetry: {"name":"Microsoft.ApplicationInsights.Dev.00000000-0000-0000-0000-000000000000.Metric", "time":"2019-12-28T00:54:19.0000000Z",
"ikey":"00000000-0000-0000-0000-000000000000",
"tags":{"ai.application.ver":"1.0.0.0",
"ai.cloud.roleInstance":"Test-Computer-Name",
"ai.internal.sdkVersion":"m-agg2c:2.12.0-21496",
"ai.internal.nodeName":"Test-Computer-Name"},
"data":{"baseType":"MetricData",
"baseData":{"ver":2,"metrics":[{"name":"ComputersSold",
"kind":"Aggregation",
"value":1722,
"count":41,
"min":42,
"max":42,
"stdDev":0}],
"properties":{"_MS.AggregationIntervalMs":"42000",
"DeveloperMode":"true"}}}}
```

Este único elemento de telemetría representa un agregado de 41 medidas de métricas distintas. Dado que se envía el mismo valor una y otra vez, tenemos una *desviación estándar (stDev)* de 0, con idénticos valores *máximo (max)* y *mínimo (min)* . La propiedad *value* representa la suma de todos los valores individuales que se agregaron.

> [!NOTE]
> GetMetric no admite el seguimiento del último valor (es decir, "gauge") ni el seguimiento de histogramas o distribuciones.

Si examinamos el recurso de Application Insights en la experiencia de Log Analytics, este elemento de telemetría individual tendría el siguiente aspecto:

![Vista de consultas de Log Analytics](./media/get-metric/log-analytics.png)

> [!NOTE]
> Aunque el elemento de telemetría sin procesar no contenía una propiedad ni campo de suma explícitos, se crea uno automáticamente después de la ingesta. En este caso, las propiedades `value` y `valueSum` representan lo mismo.

También puede acceder a los datos de telemetría de la métrica personalizada en la sección [_Métricas_](../essentials/metrics-charts.md) del portal. Tanto para [una métrica basada en registros como para una métrica personalizada](pre-aggregated-metrics-log-metrics.md). (La captura de pantalla siguiente es un ejemplo de métrica basada en registros). ![Vista del Explorador de métricas](./media/get-metric/metrics-explorer.png)

### <a name="caching-metric-reference-for-high-throughput-usage"></a>Almacenamiento en caché de referencias de métricas para uso de alto rendimiento

En algunos casos, los valores de las métricas se observan con mucha frecuencia. Por ejemplo, un servicio de alto rendimiento que procesa 500 solicitudes por segundo puede querer emitir 20 métricas de datos de telemetría por cada solicitud. Esto significa un seguimiento de 10 000 valores por segundo. En dichos escenarios de alto rendimiento, es posible que los usuarios necesiten ayudar al SDK evitando algunas búsquedas.

Por ejemplo, en este caso, en el ejemplo anterior se realizó una búsqueda de un identificador de la métrica "ComputersSold" y, a continuación, se realizó un seguimiento de un valor observado de 42. En su lugar, el identificador se puede almacenar en caché para las invocaciones de varios seguimientos:

```csharp
//...

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            // This is where the cache is stored to handle faster lookup
            Metric computersSold = _telemetryClient.GetMetric("ComputersSold");
            while (!stoppingToken.IsCancellationRequested)
            {

                computersSold.TrackValue(42);

                computersSold.TrackValue(142);

                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
                await Task.Delay(50, stoppingToken);
            }
        }

```

Además de almacenar en caché el identificador de la métrica, en el ejemplo anterior también se redujo `Task.Delay` a 50 milisegundos para que el bucle se ejecute con más frecuencia, lo que generaba 772 invocaciones a `TrackValue()`.

## <a name="multi-dimensional-metrics"></a>Métricas multidimensionales

Los ejemplos de la sección anterior muestran métricas de dimensión cero. Las métricas también pueden ser multidimensionales. Actualmente se admiten hasta 10 dimensiones.

 Este es un ejemplo de cómo crear una métrica de una dimensión:

```csharp
//...

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            // This is an example of a metric with a single dimension.
            // FormFactor is the name of the dimension.
            Metric computersSold= _telemetryClient.GetMetric("ComputersSold", "FormFactor");

            while (!stoppingToken.IsCancellationRequested)
            {
                // The number of arguments (dimension values)
                // must match the number of dimensions specified while GetMetric.
                // Laptop, Tablet, etc are values for the dimension "FormFactor"
                computersSold.TrackValue(42, "Laptop");
                computersSold.TrackValue(20, "Tablet");
                computersSold.TrackValue(126, "Desktop");


                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
                await Task.Delay(50, stoppingToken);
            }
        }

```

La ejecución de este código durante al menos 60 segundos dará como resultado que se envíen tres elementos de telemetría distintos a Azure, cada uno de los cuales representa la agregación de uno de los tres factores de forma. Como antes, puede examinarlos en la vista de Log Analytics:

![Vista de Log Analytics de una métrica multidimensional](./media/get-metric/log-analytics-multi-dimensional.png)

Así como en la experiencia del Explorador de métricas:

![Métricas personalizadas](./media/get-metric/custom-metrics.png)

Sin embargo, observará que no se puede dividir la métrica por la nueva dimensión personalizada ni ver la dimensión personalizada con la vista de métricas:

![Compatibilidad con la división](./media/get-metric/splitting-support.png)

De forma predeterminada, las métricas multidimensionales dentro de la experiencia del Explorador de métricas no están activadas en recursos de Application Insights.

### <a name="enable-multi-dimensional-metrics"></a>Habilitación de las métricas multidimensionales

Para habilitar las métricas multidimensionales para un recurso de Application Insights, seleccione **Uso y costos estimados** > **Métricas personalizadas** > **Habilitar la creación de alertas sobre las dimensiones de las métricas personalizadas** > **Aceptar**. [Aquí](pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation) puede encontrar más detalles al respecto.

Una vez que haya realizado ese cambio y haya enviado nuevos datos de telemetría multidimensionales, dispondrá de la opción **Aplicar división**.

> [!NOTE]
> Solo se almacenarán las dimensiones de las métricas enviadas después de activar la característica en el portal.

![Aplicación de la división](./media/get-metric/apply-splitting.png)

Y para ver las agregaciones de métricas para cada dimensión _FormFactor_:

![Factores de forma](./media/get-metric/formfactor.png)

### <a name="how-to-use-metricidentifier-when-there-are-more-than-three-dimensions&quot;></a>Uso de MetricIdentifier cuando hay más de tres dimensiones

En la actualidad se admiten 10 dimensiones, pero si hay más de tres dimensiones se requiere el uso de `MetricIdentifier`:

```csharp
// Add &quot;using Microsoft.ApplicationInsights.Metrics;&quot; to use MetricIdentifier
// MetricIdentifier id = new MetricIdentifier(&quot;[metricNamespace]&quot;,&quot;[metricId],&quot;[dim1]&quot;,&quot;[dim2]&quot;,&quot;[dim3]&quot;,&quot;[dim4]&quot;,&quot;[dim5]");
MetricIdentifier id = new MetricIdentifier("CustomMetricNamespace","ComputerSold", "FormFactor", "GraphicsCard", "MemorySpeed", "BatteryCapacity", "StorageCapacity");
Metric computersSold  = _telemetryClient.GetMetric(id);
computersSold.TrackValue(110,"Laptop", "Nvidia", "DDR4", "39Wh", "1TB");
```

## <a name="custom-metric-configuration"></a>Configuración de métricas personalizadas

Si desea modificar la configuración de la métrica, debe hacerlo donde se inicializa la métrica.

### <a name="special-dimension-names"></a>Nombres de dimensión especiales

Las métricas no usan el contexto de telemetría del elemento `TelemetryClient` que se usa para acceder a ellas, los nombres de dimensión especiales disponibles como constantes en la clase `MetricDimensionNames` son la mejor solución alternativa para esta limitación.

Los agregados de métricas enviados por la siguiente métrica "Tamaño de solicitud de operación especial" **no** tienen establecido `Context.Operation.Name` en "Operación especial". En su lugar, `TrackMetric()` o cualquier otro TrackXXX () tendrán establecido correctamente `OperationName` en "Operación especial".

``` csharp
        //...
        TelemetryClient specialClient;
        private static int GetCurrentRequestSize()
        {
            // Do stuff
            return 1100;
        }
        int requestSize = GetCurrentRequestSize()

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                //...
                specialClient.Context.Operation.Name = "Special Operation";
                specialClient.GetMetric("Special Operation Request Size").TrackValue(requestSize);
                //...
            }
                   
        }
```

En este caso, use los nombres de dimensión especiales que aparecen en la clase `MetricDimensionNames` para especificar los valores de `TelemetryContext`.

Por ejemplo, cuando el agregado de métricas resultante de la siguiente instrucción se envía al punto de conexión en la nube de Application Insights, el campo de datos `Context.Operation.Name` estará establecido en "Operación especial":

```csharp
_telemetryClient.GetMetric("Request Size", MetricDimensionNames.TelemetryContext.Operation.Name).TrackValue(requestSize, "Special Operation");
```

Los valores de esta dimensión especial se copiarán en `TelemetryContext` y no se utilizarán como una dimensión "normal". Si también desea mantener una dimensión de operación para la exploración normal de métricas, debe crear una dimensión independiente para ese propósito:

```csharp
_telemetryClient.GetMetric("Request Size", "Operation Name", MetricDimensionNames.TelemetryContext.Operation.Name).TrackValue(requestSize, "Special Operation", "Special Operation");
```

### <a name="dimension-and-time-series-capping"></a>Límites de las dimensiones y las series temporales

 Para evitar que el subsistema de telemetría agote accidentalmente los recursos, puede controlar el número máximo de series de datos por métrica. Los límites predeterminados son no más de 1 000 series de datos totales por métrica y no más de 100 valores diferentes por dimensión.

 En el contexto de los límites de dimensiones y series temporales, utilice `Metric.TrackValue(..)` para asegurarse de que se observan los límites. Si se han alcanzado los límites, `Metric.TrackValue(..)` devolverá "False" y no se realizará seguimiento del valor. De lo contrario, devolverá "True". Esto resulta útil si los datos de una métrica se originan a partir de los datos proporcionados por el usuario.

El constructor `MetricConfiguration` toma algunas opciones para administrar series diferentes dentro de la métrica respectiva y un objeto de una clase que implementa `IMetricSeriesConfiguration`, que especifica el comportamiento de agregación para cada serie individual de la métrica:

``` csharp
var metConfig = new MetricConfiguration(seriesCountLimit: 100, valuesPerDimensionLimit:2,
                new MetricSeriesConfigurationForMeasurement(restrictToUInt32Values: false));

Metric computersSold = _telemetryClient.GetMetric("ComputersSold", "Dimension1", "Dimension2", metConfig);

// Start tracking.
computersSold.TrackValue(100, "Dim1Value1", "Dim2Value1");
computersSold.TrackValue(100, "Dim1Value1", "Dim2Value2");

// The following call gives 3rd unique value for dimension2, which is above the limit of 2.
computersSold.TrackValue(100, "Dim1Value1", "Dim2Value3");
// The above call does not track the metric, and returns false.
```

* `seriesCountLimit` es el número máximo de series temporales de datos que puede contener una métrica. Una vez alcanzado este límite, las llamadas a `TrackValue()` que normalmente generarían una nueva serie devolverán false.
* `valuesPerDimensionLimit` limita el número de valores distintos por dimensión de forma similar.
* `restrictToUInt32Values` determina si solo se debe realizar el seguimiento de los valores enteros no negativos.

Este es un ejemplo de cómo enviar un mensaje para saber si se han superado los límites:

```csharp
if (! computersSold.TrackValue(100, "Dim1Value1", "Dim2Value3"))
{
// Add "using Microsoft.ApplicationInsights.DataContract;" to use SeverityLevel.Error
_telemetryClient.TrackTrace("Metric value not tracked as value of one of the dimension exceeded the cap. Revisit the dimensions to ensure they are within the limits",
SeverityLevel.Error);
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Más información ](./worker-service.md) sobre la supervisión de aplicaciones de servicio de trabajo.
* Más información sobre las [métricas basadas en registros y agregadas previamente](./pre-aggregated-metrics-log-metrics.md).
* [Explorador de métricas](../essentials/metrics-getting-started.md)
* Cómo habilitar Application Insights para [aplicaciones ASP.NET Core](asp-net-core.md)
* Cómo habilitar Application Insights para [aplicaciones ASP.NET](asp-net.md)
