---
title: Diagnóstico de errores y excepciones con Application Insights
description: Capture las excepciones de las aplicaciones ASP.NET junto con la telemetría de solicitudes.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 05/19/2021
ms.openlocfilehash: e992fcd16b678a89ae56ceaea4bd7d4ffb85c4fc
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110455087"
---
# <a name="diagnose-exceptions-in-web-apps-with-application-insights"></a>Diagnóstico de excepciones en aplicaciones web con Application Insights

Las excepciones en las aplicaciones web se pueden notificar con [Application Insights](./app-insights-overview.md). Puede correlacionar las solicitudes con error con excepciones y otros eventos en el cliente y en el servidor, de modo que pueda diagnosticar rápidamente las causas. En este artículo, aprenderá a configurar informes de excepciones, notificar excepciones explícitamente, diagnosticar errores, etc.

## <a name="set-up-exception-reporting"></a>Configuración de informes de excepciones

Puede configurar Application Insights para notificar las excepciones que se producen en el servidor o en el cliente. En función de la plataforma de la que dependa la aplicación, necesitará la extensión o el SDK adecuados.

### <a name="server-side"></a>En el servidor

Para que se notifiquen las excepciones desde la aplicación de servidor, tenga en cuenta los siguientes escenarios:

  * **Aplicaciones web de Azure**: agregue la [extensión de Application Insights](./azure-web-apps.md).
  * **Aplicaciones hospedadas en IIS en VM de Azure y conjunto de escalado de máquinas virtuales de Azure**: agregue la [extensión de supervisión de aplicaciones](./azure-vm-vmss-apps.md).
  * Instale el [SDK de Application Insights](./asp-net.md) en su código de aplicación.
  * **Servidores web de IIS**: ejecute el [agente de Application Insights](./monitor-performance-live-website-now.md).
  * **Aplicaciones web de Java**: habilite el [agente Java](./java-in-process-agent.md).

### <a name="client-side"></a>En el cliente

El SDK de JavaScript proporciona la capacidad de generar informes de cliente de las excepciones que se producen en exploradores web. Para configurar informes de excepciones en el cliente, vea [Application Insights para páginas web](./javascript.md).

### <a name="application-frameworks"></a>Marcos de aplicaciones

Con algunos marcos de aplicaciones, se requiere un poco más de configuración. Considere la posibilidad de utilizar las siguientes tecnologías:

  * [Formularios web](#web-forms)
  * [MVC](#mvc)
  * [API web 1.*](#web-api-1x)
  * [API web 2.*](#web-api-2x)
  * [WCF](#wcf)

> [!IMPORTANT]
> Este artículo se centra específicamente en aplicaciones de .NET Framework desde la perspectiva de un ejemplo de código. Algunos de los métodos que funcionan en .NET Framework son obsoletos en el SDK de .NET Core. Para obtener más información, consulte la [documentación del SDK de .NET Core](./asp-net-core.md) al compilar aplicaciones con .NET Core.

## <a name="diagnose-exceptions-using-visual-studio"></a>Diagnóstico de excepciones mediante Visual Studio

Abre la solución de aplicación en Visual Studio. Ejecute la aplicación, bien en el servidor o en la máquina de desarrollo mediante <kbd>F5</kbd>. Vuelva a crear la excepción.

Abra la ventana de telemetría **Búsqueda de Application Insights** en Visual Studio. Durante la depuración, seleccione el menú desplegable de **Application Insights**.

![Haga clic con el botón derecho en el proyecto y seleccione Application Insights, Abrir.](./media/asp-net-exceptions/34.png)

Haga clic en un informe de excepciones para mostrar su seguimiento de la pila. Seleccione una referencia de línea en el seguimiento de la pila para abrir el archivo de código pertinente.

Si CodeLens está habilitado, verá datos sobre las excepciones:

![Notificación de CodeLens de excepciones.](./media/asp-net-exceptions/35.png)

## <a name="diagnose-failures-using-the-azure-portal"></a>Diagnóstico de errores mediante Azure Portal

Application Insights incluye una experiencia de administración del rendimiento de las aplicaciones (APM) seleccionada para ayudar a diagnosticar errores en las aplicaciones supervisadas. Para empezar, seleccione la opción **Errores** del menú de recursos de Application Insights situado en la sección **Investigar**.
Verá las tendencias de las tasas de errores de las solicitudes, cuántas de ellas tienen errores y a cuántos usuarios afectan. A modo de vista **general**, observará algunas de las distribuciones más útiles específicas de la operación con error seleccionada, incluidos los tres códigos de respuesta principales, los tres tipos principales de excepción y los tres tipos principales de dependencia con error.

![Vista de evaluación de prioridades de errores (pestaña Operaciones)](./media/asp-net-exceptions/failures0719.png)

Para revisar ejemplos representativos de cada uno de estos subconjuntos de operaciones, seleccione el vínculo correspondiente. En concreto, para diagnosticar excepciones, puede hacer clic en el recuento de una excepción determinada para que se muestre con la pestaña de detalles **Transacción completa**:

![Hoja de detalles Transacción completa](./media/asp-net-exceptions/end-to-end.png)

Como alternativa, en lugar de observar las excepciones de una operación específica con errores, puede empezar desde la vista **general** de excepciones y cambiar a la pestaña **Excepciones** en la parte superior. Aquí puede ver todas las excepciones que se recopilan para la aplicación supervisada.

## <a name="custom-tracing-and-log-data"></a>Personalización del seguimiento y del registro de datos

Para obtener datos de diagnóstico específicos de su aplicación, puede insertar código para enviar sus propios datos de telemetría. Sus datos de telemetría o registro personalizados aparecen en la búsqueda de diagnóstico junto con la solicitud, la vista de página y otros datos que se recopilan automáticamente.

Con <xref:Microsoft.ApplicationInsights.TelemetryClient?displayProperty=fullName>, tiene varias API disponibles:

* <xref:Microsoft.ApplicationInsights.TelemetryClient.TrackEvent%2A?displayProperty=nameWithType> normalmente se usa para supervisar patrones de uso, pero los datos que envía también aparecen en **Eventos personalizados** en la búsqueda de diagnósticos. Los eventos tienen nombre y pueden llevar propiedades de cadena y métricas numéricas en las que puede [filtrar las búsquedas de diagnósticos](./diagnostic-search.md).
* <xref:Microsoft.ApplicationInsights.TelemetryClient.TrackTrace%2A?displayProperty=nameWithType> le permite enviar datos más grandes, como la información de POST.
* <xref:Microsoft.ApplicationInsights.TelemetryClient.TrackException%2A?displayProperty=nameWithType> envía detalles de excepción, como seguimientos de pila a Application Insights.

Para ver estos eventos, abra [Buscar](./diagnostic-search.md) en el menú de la izquierda, seleccione el menú desplegable **Tipos de evento** y elija **Evento personalizado**, **Seguimiento** o **Excepción**.

![Obtener detalles](./media/asp-net-exceptions/customevents.png)

> [!NOTE]
> Si la aplicación genera mucha telemetría, el módulo de muestreo adaptable reducirá automáticamente el volumen que se envía al portal mediante el envío de únicamente una fracción representativa de eventos. Los eventos que forman parte de la misma operación se seleccionarán o se anulará su selección como grupo, por lo que puede navegar entre los eventos relacionados. Para más información, consulte [Muestreo en Application Insights](./sampling.md).

### <a name="how-to-see-request-post-data"></a>Visualización de los datos de solicitud POST

Los detalles de la solicitud no incluyen los datos enviados a la aplicación en una llamada a POST. Para que se notifiquen estos datos:

* [Instale el SDK](./asp-net.md) en su proyecto de aplicación.
* Inserte código en la aplicación para llamar a [Microsoft.ApplicationInsights.TrackTrace()](./api-custom-events-metrics.md#tracktrace). Envíe los datos de POST en el parámetro de mensaje. Hay un límite en cuanto al tamaño permitido, así que debe intentar enviar únicamente los datos fundamentales.
* Cuando investigue una solicitud con error, busque los seguimientos asociados.

## <a name="capturing-exceptions-and-related-diagnostic-data"></a><a name="exceptions"></a> Captura de excepciones y datos de diagnóstico relacionados
En primer lugar, no verá en el portal todas las excepciones que provocan errores en su aplicación. Verá las excepciones del explorador (si usa el [SDK de JavaScript](./javascript.md) en sus páginas web). Pero la mayoría de las excepciones de servidor las detecta IIS y debe escribir algo de código para verlas.

Puede:

* **Registrar excepciones explícitamente** insertando código en los controladores de excepciones para notificar las excepciones.
* **Capturar excepciones automáticamente** configurando su marco de ASP.NET. Las adiciones necesarias son diferentes para los distintos tipos de marco.

## <a name="reporting-exceptions-explicitly"></a>Notificación de excepciones explícitamente

La manera más sencilla consiste en insertar una llamada a `trackException()` en un controlador de excepciones.

```javascript
try
{
    // ...
}
catch (ex)
{
    appInsights.trackException(ex, "handler loc",
    {
        Game: currentGame.Name,
        State: currentGame.State.ToString()
    });
}
```

```csharp
var telemetry = new TelemetryClient();

try
{
    // ...
}
catch (Exception ex)
{
    var properties = new Dictionary<string, string>
    {
        ["Game"] = currentGame.Name
    };

    var measurements = new Dictionary<string, double>
    {
        ["Users"] = currentGame.Users.Count
    };

    // Send the exception telemetry:
    telemetry.TrackException(ex, properties, measurements);
}
```

```VB
Dim telemetry = New TelemetryClient

Try
    ' ...
Catch ex as Exception
    ' Set up some properties:
    Dim properties = New Dictionary (Of String, String)
    properties.Add("Game", currentGame.Name)

    Dim measurements = New Dictionary (Of String, Double)
    measurements.Add("Users", currentGame.Users.Count)

    ' Send the exception telemetry:
    telemetry.TrackException(ex, properties, measurements)
End Try
```

Los parámetros de las propiedades y las medidas son opcionales, pero son útiles para [filtrar y agregar información](./diagnostic-search.md) adicional. Por ejemplo, si tiene una aplicación que se puede ejecutar varios juegos, podría buscar todos los informes de excepción relacionados con un juego en particular. Puede agregar tantos elementos como desee para cada diccionario.

## <a name="browser-exceptions"></a>Excepciones de explorador

Se notifican la mayoría de las excepciones de explorador.

Si la página web incluye archivos de script de redes de entrega de contenido o de otros dominios, asegúrese de que la etiqueta de script tenga el atributo `crossorigin="anonymous"`, y que el servidor envíe [encabezados CORS](https://enable-cors.org/). Esto le permitirá obtener un seguimiento de pila y los detalles de las excepciones no controladas de JavaScript de estos recursos.

## <a name="reuse-your-telemetry-client"></a>Reutilización del cliente de telemetría

> [!NOTE]
> Se recomienda crear una instancia de `TelemetryClient` una vez y reutilizarla durante todo el ciclo de vida de una aplicación.

Con la [inserción de dependencias (DI) en .NET](/dotnet/core/extensions/dependency-injection), el SDK de .NET adecuado y la configuración correcta de Application Insights para la inserción de dependencias, puede requerir <xref:Microsoft.ApplicationInsights.TelemetryClient> como un parámetro de constructor.

```csharp
public class ExampleController : ApiController
{
    private readonly TelemetryClient _telemetryClient;

    public ExampleController(TelemetryClient telemetryClient)
    {
        _telemetryClient = telemetryClient;
    }
}
```

En el ejemplo anterior, el parámetro `TelemetryClient` se inserta en la clase `ExampleController`.

## <a name="web-forms"></a>Formularios web

Para los formularios web, el módulo HTTP podrá recopilar las excepciones cuando no haya ningún redireccionamiento configurado con `CustomErrors`. No obstante, si tiene redireccionamientos activos, agregue las siguientes líneas a la función `Application_Error` en *Global.asax.cs*.

```csharp
void Application_Error(object sender, EventArgs e)
{
    if (HttpContext.Current.IsCustomErrorEnabled &&
        Server.GetLastError () != null)
    {
        _telemetryClient.TrackException(Server.GetLastError());
    }
}
```

En el ejemplo anterior, `_telemetryClient` es una variable de ámbito de clase de tipo <xref:Microsoft.ApplicationInsights.TelemetryClient>.

## <a name="mvc"></a>MVC

A partir del SDK web de Application Insights versión 2.6 (beta3 y posterior), Application Insights recopila excepciones no controladas producidas automáticamente en los métodos de controladores MVC 5+. Si anteriormente ha agregado un controlador personalizado para realizar el seguimiento de tales excepciones, puede quitarlo para impedir el seguimiento duplicado de las excepciones.

Hay una serie de escenarios en los que un filtro de excepciones no puede controlar correctamente los errores cuando se producen excepciones:

* Desde constructores de controlador.
* Desde controladores de mensajes.
* Durante el enrutamiento.
* Durante la serialización del contenido de respuesta.
* Durante el inicio de la aplicación.
* En tareas en segundo plano.

Es necesario seguir realizando el seguimiento de todas las excepciones *controladas* por la aplicación manualmente.
Las excepciones no controladas que se originan en los controladores dan lugar normalmente a respuestas 500 "Error interno del servidor". Si dicha respuesta se creó manualmente como resultado de una excepción controlada (o sin ninguna excepción en absoluto), se realiza su seguimiento en la telemetría de la solicitud correspondiente `ResultCode` 500; sin embargo, el SDK de Application Insights no puede realizar el seguimiento de la excepción correspondiente.

### <a name="prior-versions-support"></a>Compatibilidad con versiones anteriores

Si usa MVC 4 (y anterior) del SDK web de Application Insights 2.5 (y anterior), consulte los siguientes ejemplos para realizar el seguimiento de las excepciones.

Si la configuración de [CustomErrors](/previous-versions/dotnet/netframework-4.0/h0hfz6fc(v=vs.100)) es `Off`, entonces habrá excepciones disponibles para que las recopile el [módulo HTTP](/previous-versions/dotnet/netframework-3.0/ms178468(v=vs.85)). Sin embargo, si es `RemoteOnly` (valor predeterminado), o `On`, la excepción se desactivará y no estará disponible para que Application Insights la recopile automáticamente. Para corregir este problema, invalide la [clase System.Web.Mvc.HandleErrorAttribute](/dotnet/api/system.web.mvc.handleerrorattribute) y aplique la clase invalidada como se muestra a continuación para las diferentes versiones de MVC ([origen de GitHub](https://github.com/AppInsightsSamples/Mvc2UnhandledExceptions/blob/master/MVC2App/Controllers/AiHandleErrorAttribute.cs)):

```csharp
using System;
using System.Web.Mvc;
using Microsoft.ApplicationInsights;

namespace MVC2App.Controllers
{
    [AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, Inherited = true, AllowMultiple = true)]
    public class AiHandleErrorAttribute : HandleErrorAttribute
    {
        public override void OnException(ExceptionContext filterContext)
        {
            if (filterContext != null && filterContext.HttpContext != null && filterContext.Exception != null)
            {
                //If customError is Off, then AI HTTPModule will report the exception
                if (filterContext.HttpContext.IsCustomErrorEnabled)
                {   //or reuse instance (recommended!). see note above
                    var ai = new TelemetryClient();
                    ai.TrackException(filterContext.Exception);
                }
            }
            base.OnException(filterContext);
        }
    }
}
```

#### <a name="mvc-2"></a>MVC 2

Sustituya el atributo HandleError por el nuevo atributo en los controladores.

```csharp
    namespace MVC2App.Controllers
    {
        [AiHandleError]
        public class HomeController : Controller
        {
            // Omitted for brevity
        }
    }
```

[Ejemplo](https://github.com/AppInsightsSamples/Mvc2UnhandledExceptions)

#### <a name="mvc-3"></a>MVC 3

Registrar `AiHandleErrorAttribute` como filtro global de *Global.asax.cs*:

```csharp
public class MyMvcApplication : System.Web.HttpApplication
{
    public static void RegisterGlobalFilters(GlobalFilterCollection filters)
    {
        filters.Add(new AiHandleErrorAttribute());
    }
}
```

[Ejemplo](https://github.com/AppInsightsSamples/Mvc3UnhandledExceptionTelemetry)

#### <a name="mvc-4-mvc5"></a>MVC 4, MVC5

Registrar `AiHandleErrorAttribute` como filtro global de *FilterConfig.cs*:

```csharp
public class FilterConfig
{
    public static void RegisterGlobalFilters(GlobalFilterCollection filters)
    {
        // Default replaced with the override to track unhandled exceptions
        filters.Add(new AiHandleErrorAttribute());
    }
}
```

[Ejemplo](https://github.com/AppInsightsSamples/Mvc5UnhandledExceptionTelemetry)

## <a name="web-api"></a>API Web

A partir del SDK web de Application Insights versión 2.6 (beta3 y posterior), Application Insights recopila excepciones no controladas producidas automáticamente en los métodos de controladores para WebAPI 2+. Si anteriormente ha agregado un controlador personalizado para realizar el seguimiento de tales excepciones (como se describe en los ejemplos siguientes), puede quitarlo para impedir el seguimiento de excepciones por duplicado.

Hay un número de casos que los filtros de excepciones no pueden procesar. Por ejemplo:

* Excepciones iniciadas por constructores del controlador.
* Excepciones iniciadas por controladores de mensajes.
* Excepciones iniciadas durante el enrutamiento.
* Excepciones iniciadas durante la serialización del contenido de respuesta.
* Excepción que se produce durante el inicio de la aplicación.
* Excepción que se produce en tareas en segundo plano.

Es necesario seguir realizando el seguimiento de todas las excepciones *controladas* por la aplicación manualmente.
Las excepciones no controladas que se originan en los controladores dan lugar normalmente a respuestas 500 "Error interno del servidor". Si dicha respuesta se creó manualmente como resultado de una excepción controlada (o sin ninguna excepción en absoluto), se realiza su seguimiento en la telemetría de la solicitud correspondiente `ResultCode` 500; sin embargo, el SDK de Application Insights no puede realizar el seguimiento de la excepción correspondiente.

### <a name="prior-versions-support"></a>Compatibilidad con versiones anteriores

Si usa WebAPI 1 (y anterior) del SDK web de Application Insights 2.5 (y anterior), consulte los siguientes ejemplos para realizar el seguimiento de las excepciones.

#### <a name="web-api-1x"></a>Web API 1.x

Invalidar `System.Web.Http.Filters.ExceptionFilterAttribute`:

```csharp
using System.Web.Http.Filters;
using Microsoft.ApplicationInsights;

namespace WebAPI.App_Start
{
    public class AiExceptionFilterAttribute : ExceptionFilterAttribute
    {
    public override void OnException(HttpActionExecutedContext actionExecutedContext)
    {
        if (actionExecutedContext != null && actionExecutedContext.Exception != null)
        {  //or reuse instance (recommended!). see note above
            var ai = new TelemetryClient();
            ai.TrackException(actionExecutedContext.Exception);
        }
        base.OnException(actionExecutedContext);
    }
    }
}
```

Podría agregar este atributo invalidado a controladores específicos o agregarlo a la configuración de filtros globales en la clase `WebApiConfig`:

```csharp
using System.Web.Http;
using WebApi1.x.App_Start;

namespace WebApi1.x
{
    public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {
            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional });
    
            // ...
            config.EnableSystemDiagnosticsTracing();
    
            // Capture exceptions for Application Insights:
            config.Filters.Add(new AiExceptionFilterAttribute());
        }
    }
}
```

[Ejemplo](https://github.com/AppInsightsSamples/WebApi_1.x_UnhandledExceptions)

#### <a name="web-api-2x"></a>Web API 2.x

Agregar una implementación de `IExceptionLogger`:

```csharp
using System.Web.Http.ExceptionHandling;
using Microsoft.ApplicationInsights;

namespace ProductsAppPureWebAPI.App_Start
{
    public class AiExceptionLogger : ExceptionLogger
    {
        public override void Log(ExceptionLoggerContext context)
        {
            if (context != null && context.Exception != null)
            {
                //or reuse instance (recommended!). see note above
                var ai = new TelemetryClient();
                ai.TrackException(context.Exception);
            }
            base.Log(context);
        }
    }
}
```

Agregue lo siguiente a los servicios en WebApiConfig:

```csharp
using System.Web.Http;
using System.Web.Http.ExceptionHandling;
using ProductsAppPureWebAPI.App_Start;

namespace WebApi2WithMVC
{
    public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {
            // Web API configuration and services
    
            // Web API routes
            config.MapHttpAttributeRoutes();
    
            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional });

            config.Services.Add(typeof(IExceptionLogger), new AiExceptionLogger());
        }
    }
}
```

[Ejemplo](https://github.com/AppInsightsSamples/WebApi_2.x_UnhandledExceptions)

Como alternativa, puede:

1. Sustituir el único ExceptionHandler por una implementación personalizada de IExceptionHandler. Solo se llama a este controlador de excepciones cuando el marco es aún capaz de seleccionar el mensaje de respuesta que se enviará (no cuando se anula la conexión, por ejemplo)
2. Los filtros de excepciones (como se describe en la sección sobre controladores de Web API 1.x anteriores) no se llaman en todos los casos.

## <a name="wcf"></a>WCF

Agregue una clase que extienda el atributo y que implemente IErrorHandler y IServiceBehavior.

```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.ServiceModel.Description;
    using System.ServiceModel.Dispatcher;
    using System.Web;
    using Microsoft.ApplicationInsights;

    namespace WcfService4.ErrorHandling
    {
      public class AiLogExceptionAttribute : Attribute, IErrorHandler, IServiceBehavior
      {
        public void AddBindingParameters(ServiceDescription serviceDescription,
            System.ServiceModel.ServiceHostBase serviceHostBase,
            System.Collections.ObjectModel.Collection<ServiceEndpoint> endpoints,
            System.ServiceModel.Channels.BindingParameterCollection bindingParameters)
        {
        }

        public void ApplyDispatchBehavior(ServiceDescription serviceDescription,
            System.ServiceModel.ServiceHostBase serviceHostBase)
        {
            foreach (ChannelDispatcher disp in serviceHostBase.ChannelDispatchers)
            {
                disp.ErrorHandlers.Add(this);
            }
        }

        public void Validate(ServiceDescription serviceDescription,
            System.ServiceModel.ServiceHostBase serviceHostBase)
        {
        }

        bool IErrorHandler.HandleError(Exception error)
        {//or reuse instance (recommended!). see note above
            var ai = new TelemetryClient();

            ai.TrackException(error);
            return false;
        }

        void IErrorHandler.ProvideFault(Exception error,
            System.ServiceModel.Channels.MessageVersion version,
            ref System.ServiceModel.Channels.Message fault)
        {
        }
      }
    }
```

Agregue el atributo a las implementaciones de servicio:

```csharp
namespace WcfService4
{
    [AiLogException]
    public class Service1 : IService1
    {
        // Omitted for brevity
    }
}
```

[Ejemplo](https://github.com/AppInsightsSamples/WCFUnhandledExceptions)

## <a name="exception-performance-counters"></a>Contadores de rendimiento de excepciones

Si tiene [instalado el agente de Application Insights](./monitor-performance-live-website-now.md) en el servidor, puede obtener un gráfico de la tasa de excepciones, medida por .NET. Esto incluye las excepciones de .NET, tanto controladas como no controladas.

Abra una pestaña del Explorador de métricas, agregue un nuevo gráfico y seleccione **Tasa de excepciones**, que aparece debajo de Contadores de rendimiento.

.NET framework calcula la tasa contando el número de excepciones producidas en un intervalo y dividiéndolo por la duración del intervalo de tiempo.

Esto es diferente del recuento de "Excepciones" calculado por el portal de Application Insights contando los informes de TrackException. Los intervalos de muestreo son diferentes y el SDK no envía informes de TrackException para todas las excepciones, controladas y no controladas.

## <a name="next-steps"></a>Pasos siguientes

* [Supervisar REST, SQL y otras llamadas a las dependencias](./asp-net-dependencies.md)
* [Supervisar los tiempos de carga de página, las excepciones del explorador y las llamadas AJAX](./javascript.md)
* [Supervisar los contadores de rendimiento](./performance-counters.md)
