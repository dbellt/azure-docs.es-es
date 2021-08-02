---
title: Configuración de la supervisión para ASP.NET con Azure Application Insights | Microsoft Docs
description: Configure las herramientas de análisis del rendimiento, la disponibilidad y de comportamiento de usuarios para un sitio web de ASP.NET, hospedado localmente o en Azure.
ms.topic: conceptual
ms.date: 09/30/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: e7cc1038e662950e8f7207d3cf4d9bf9e45f90e1
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110097271"
---
# <a name="configure-application-insights-for-your-aspnet-website"></a>Configuración de Application Insights para un sitio web de ASP.NET

Este procedimiento configura una aplicación web de ASP.NET para que envíe datos de telemetría al servicio [Azure Application Insights](./app-insights-overview.md). Funciona en las aplicaciones ASP.NET que se hospedan en sus propios servidores IIS locales o en la nube. 

## <a name="prerequisites"></a>Prerrequisitos
Para agregar Application Insights al sitio web de ASP.NET, necesita:

- Instale la versión más reciente de [Visual Studio 2019 para Windows](https://www.visualstudio.com/downloads/) con las cargas de trabajo siguientes:
    - ASP.NET y desarrollo web.
    - Desarrollo de Azure

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- Cree un [recurso basado en el área de trabajo de Application Insights](create-workspace-resource.md).

> [!IMPORTANT]
> Las [cadenas de conexión](./sdk-connection-string.md?tabs=net) se recomiendan por encima de las claves de instrumentación. Las nuevas regiones de Azure **requieren** el uso de cadenas de conexión en lugar de claves de instrumentación. La cadena de conexión identifica el recurso con el que se quieren asociar los datos de telemetría. También permite modificar los puntos de conexión que va a usar el recurso como destino de la telemetría. Tiene que copiar la cadena de conexión y agregarla al código de la aplicación o a una variable de entorno.


## <a name="create-a-basic-aspnet-web-app"></a>Creación de una aplicación web de ASP.NET básica

1. Inicie Visual Studio 2019.
2. Seleccione **File (Archivo)**  > **New (Nuevo)**  > **Project (Proyecto)** .
3. Seleccione **Aplicación web ASP.NET (.NET Framework) C#** .
4. Escriba un nombre de proyecto > **Seleccione Crear**.
5. Seleccione **MVC** > **crear**. 

## <a name="add-application-insights-automatically"></a>Incorporación de Application Insights automáticamente

Esta sección le guiará a través de la incorporación automática de Application Insights a una aplicación web de ASP.NET basada en plantilla. Desde dentro del proyecto de aplicación web de ASP.NET en Visual Studio:

1. Seleccione **Add Application Insights Telemetry** (Agregar telemetría de Application Insights) > **Application Insights SDK (local)** (SDK de Application Insights [local]) > **Siguiente** > **Finalizar** > **Cerrar**.
2. Abra el archivo `ApplicationInsights.config` . 
3. Antes de la etiqueta `</ApplicationInsights>` de cierre, agregue una línea que contenga la clave de instrumentación del recurso de Application Insights.  La clave de instrumentación se puede encontrar en el panel de información general del recurso de Application Insights recién creado que creó como parte de los requisitos previos de este artículo.

    ```xml
    <InstrumentationKey>your-instrumentation-key-goes-here</InstrumentationKey>
    ```
4. Seleccione **Proyecto** > **Manage NuGet Packages** (Administrar paquetes NuGet) > **Actualizaciones** > Actualice cada paquete NuGet `Microsoft.ApplicationInsights` a la versión estable más reciente.   
5. Ejecute la aplicación seleccionando **IIS Express**. Se iniciará una aplicación ASP.NET básica. A medida que navega por las páginas en la telemetría del sitio se enviarán a Application Insights.

## <a name="add-application-insights-manually"></a>Incorporación de Application Insights manualmente

Esta sección le guiará a través de la incorporación manual de Application Insights a una aplicación web de ASP.NET basada en plantilla. En esta sección se da por supuesto que está usando una aplicación web basada en la plantilla de aplicación web MVC de ASP.NET Framework estándar.

1. Agregue los siguientes paquetes NuGet y sus dependencias al proyecto:

    - [`Microsoft.ApplicationInsights.WindowsServer`](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer)
    - [`Microsoft.ApplicationInsights.Web`](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web)
    - [`Microsoft.AspNet.TelemetryCorrelation`](https://www.nuget.org/packages/Microsoft.AspNet.TelemetryCorrelation)

2. En algunos casos, el archivo de `ApplicationInsights.config` se creará automáticamente. Si el archivo ya está presente, vaya al paso 4. Si no se crea automáticamente, tendrá que crearlo usted mismo. En el mismo nivel del proyecto que el archivo `Global.asax`, cree un nuevo archivo denominado `ApplicationInsights.config`.

3. Copie la siguiente configuración XML en el archivo recién creado:

     ```xml
     <?xml version="1.0" encoding="utf-8"?>
    <ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings">
      <TelemetryInitializers>
        <Add Type="Microsoft.ApplicationInsights.DependencyCollector.HttpDependenciesParsingTelemetryInitializer, Microsoft.AI.DependencyCollector" />
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.AzureRoleEnvironmentTelemetryInitializer, Microsoft.AI.WindowsServer" />
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.BuildInfoConfigComponentVersionTelemetryInitializer, Microsoft.AI.WindowsServer" />
        <Add Type="Microsoft.ApplicationInsights.Web.WebTestTelemetryInitializer, Microsoft.AI.Web" />
        <Add Type="Microsoft.ApplicationInsights.Web.SyntheticUserAgentTelemetryInitializer, Microsoft.AI.Web">
          <!-- Extended list of bots:
                search|spider|crawl|Bot|Monitor|BrowserMob|BingPreview|PagePeeker|WebThumb|URL2PNG|ZooShot|GomezA|Google SketchUp|Read Later|KTXN|KHTE|Keynote|Pingdom|AlwaysOn|zao|borg|oegp|silk|Xenu|zeal|NING|htdig|lycos|slurp|teoma|voila|yahoo|Sogou|CiBra|Nutch|Java|JNLP|Daumoa|Genieo|ichiro|larbin|pompos|Scrapy|snappy|speedy|vortex|favicon|indexer|Riddler|scooter|scraper|scrubby|WhatWeb|WinHTTP|voyager|archiver|Icarus6j|mogimogi|Netvibes|altavista|charlotte|findlinks|Retreiver|TLSProber|WordPress|wsr-agent|http client|Python-urllib|AppEngine-Google|semanticdiscovery|facebookexternalhit|web/snippet|Google-HTTP-Java-Client-->
          <Filters>search|spider|crawl|Bot|Monitor|AlwaysOn</Filters>
        </Add>
        <Add Type="Microsoft.ApplicationInsights.Web.ClientIpHeaderTelemetryInitializer, Microsoft.AI.Web" />
        <Add Type="Microsoft.ApplicationInsights.Web.AzureAppServiceRoleNameFromHostNameHeaderInitializer, Microsoft.AI.Web" />
        <Add Type="Microsoft.ApplicationInsights.Web.OperationNameTelemetryInitializer, Microsoft.AI.Web" />
        <Add Type="Microsoft.ApplicationInsights.Web.OperationCorrelationTelemetryInitializer, Microsoft.AI.Web" />
        <Add Type="Microsoft.ApplicationInsights.Web.UserTelemetryInitializer, Microsoft.AI.Web" />
        <Add Type="Microsoft.ApplicationInsights.Web.AuthenticatedUserIdTelemetryInitializer, Microsoft.AI.Web" />
        <Add Type="Microsoft.ApplicationInsights.Web.AccountIdTelemetryInitializer, Microsoft.AI.Web" />
        <Add Type="Microsoft.ApplicationInsights.Web.SessionTelemetryInitializer, Microsoft.AI.Web" />
      </TelemetryInitializers>
      <TelemetryModules>
        <Add Type="Microsoft.ApplicationInsights.DependencyCollector.DependencyTrackingTelemetryModule, Microsoft.AI.DependencyCollector">
          <ExcludeComponentCorrelationHttpHeadersOnDomains>
            <!-- 
            Requests to the following hostnames will not be modified by adding correlation headers.         
            Add entries here to exclude additional hostnames.
            NOTE: this configuration will be lost upon NuGet upgrade.
            -->
            <Add>core.windows.net</Add>
            <Add>core.chinacloudapi.cn</Add>
            <Add>core.cloudapi.de</Add>
            <Add>core.usgovcloudapi.net</Add>
          </ExcludeComponentCorrelationHttpHeadersOnDomains>
          <IncludeDiagnosticSourceActivities>
            <Add>Microsoft.Azure.EventHubs</Add>
            <Add>Microsoft.Azure.ServiceBus</Add>
          </IncludeDiagnosticSourceActivities>
        </Add>
        <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
          <!--
          Use the following syntax here to collect additional performance counters:
          
          <Counters>
            <Add PerformanceCounter="\Process(??APP_WIN32_PROC??)\Handle Count" ReportAs="Process handle count" />
            ...
          </Counters>
          
          PerformanceCounter must be either \CategoryName(InstanceName)\CounterName or \CategoryName\CounterName
          
          NOTE: performance counters configuration will be lost upon NuGet upgrade.
          
          The following placeholders are supported as InstanceName:
            ??APP_WIN32_PROC?? - instance name of the application process  for Win32 counters.
            ??APP_W3SVC_PROC?? - instance name of the application IIS worker process for IIS/ASP.NET counters.
            ??APP_CLR_PROC?? - instance name of the application CLR process for .NET counters.
          -->
        </Add>
        <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector" />
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.AppServicesHeartbeatTelemetryModule, Microsoft.AI.WindowsServer" />
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.AzureInstanceMetadataTelemetryModule, Microsoft.AI.WindowsServer">
          <!--
          Remove individual fields collected here by adding them to the ApplicationInsighs.HeartbeatProvider 
          with the following syntax:
          
          <Add Type="Microsoft.ApplicationInsights.Extensibility.Implementation.Tracing.DiagnosticsTelemetryModule, Microsoft.ApplicationInsights">
            <ExcludedHeartbeatProperties>
              <Add>osType</Add>
              <Add>location</Add>
              <Add>name</Add>
              <Add>offer</Add>
              <Add>platformFaultDomain</Add>
              <Add>platformUpdateDomain</Add>
              <Add>publisher</Add>
              <Add>sku</Add>
              <Add>version</Add>
              <Add>vmId</Add>
              <Add>vmSize</Add>
              <Add>subscriptionId</Add>
              <Add>resourceGroupName</Add>
              <Add>placementGroupId</Add>
              <Add>tags</Add>
              <Add>vmScaleSetName</Add>
            </ExcludedHeartbeatProperties>
          </Add>
                
          NOTE: exclusions will be lost upon upgrade.
          -->
        </Add>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.DeveloperModeWithDebuggerAttachedTelemetryModule, Microsoft.AI.WindowsServer" />
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.UnhandledExceptionTelemetryModule, Microsoft.AI.WindowsServer" />
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.UnobservedExceptionTelemetryModule, Microsoft.AI.WindowsServer">
          <!--</Add>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.FirstChanceExceptionStatisticsTelemetryModule, Microsoft.AI.WindowsServer">-->
        </Add>
        <Add Type="Microsoft.ApplicationInsights.Web.RequestTrackingTelemetryModule, Microsoft.AI.Web">
          <Handlers>
            <!-- 
            Add entries here to filter out additional handlers: 
            
            NOTE: handler configuration will be lost upon NuGet upgrade.
            -->
            <Add>Microsoft.VisualStudio.Web.PageInspector.Runtime.Tracing.RequestDataHttpHandler</Add>
            <Add>System.Web.StaticFileHandler</Add>
            <Add>System.Web.Handlers.AssemblyResourceLoader</Add>
            <Add>System.Web.Optimization.BundleHandler</Add>
            <Add>System.Web.Script.Services.ScriptHandlerFactory</Add>
            <Add>System.Web.Handlers.TraceHandler</Add>
            <Add>System.Web.Services.Discovery.DiscoveryRequestHandler</Add>
            <Add>System.Web.HttpDebugHandler</Add>
          </Handlers>
        </Add>
        <Add Type="Microsoft.ApplicationInsights.Web.ExceptionTrackingTelemetryModule, Microsoft.AI.Web" />
        <Add Type="Microsoft.ApplicationInsights.Web.AspNetDiagnosticTelemetryModule, Microsoft.AI.Web" />
      </TelemetryModules>
      <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights" />
      <TelemetrySinks>
        <Add Name="default">
          <TelemetryProcessors>
            <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryProcessor, Microsoft.AI.PerfCounterCollector" />
            <Add Type="Microsoft.ApplicationInsights.Extensibility.AutocollectedMetricsExtractor, Microsoft.ApplicationInsights" />
            <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
              <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
              <ExcludedTypes>Event</ExcludedTypes>
            </Add>
            <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
              <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
              <IncludedTypes>Event</IncludedTypes>
            </Add>
          </TelemetryProcessors>
          <TelemetryChannel Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.ServerTelemetryChannel, Microsoft.AI.ServerTelemetryChannel" />
        </Add>
      </TelemetrySinks>
      <!-- 
        Learn more about Application Insights configuration with ApplicationInsights.config here: 
        http://go.microsoft.com/fwlink/?LinkID=513840
      -->
      <InstrumentationKey>your-instrumentation-key-here</InstrumentationKey>
    </ApplicationInsights>
     ```

4. Antes de la etiqueta `</ApplicationInsights>` de cierre, agregue la clave de instrumentación del recurso de Application Insights.  La clave de instrumentación se puede encontrar en el panel de información general del recurso de Application Insights recién creado que creó como parte de los requisitos previos de este artículo.

    ```xml
    <InstrumentationKey>your-instrumentation-key-goes-here</InstrumentationKey>
    ```

5. En el mismo nivel del proyecto que el archivo `ApplicationInsights.config`, cree una carpeta llamada `ErrorHandler` con un nuevo archivo de C# denominado `AiHandleErrorAttribute.cs`. El contenido del archivo tendrá el siguiente aspecto:

    ```csharp
    using System;
    using System.Web.Mvc;
    using Microsoft.ApplicationInsights;
    
    namespace WebApplication10.ErrorHandler //namespace will vary based on your project name
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
                    {   
                        var ai = new TelemetryClient();
                        ai.TrackException(filterContext.Exception);
                    } 
                }
                base.OnException(filterContext);
            }
        }
    }
    
    ```

6. En la carpeta `App_Start`, abra el archivo `FilterConfig.cs` y cámbielo para que coincida con el ejemplo:

    ```csharp
    using System.Web;
    using System.Web.Mvc;
    
    namespace WebApplication10 //Namespace will vary based on project name
    {
        public class FilterConfig
        {
            public static void RegisterGlobalFilters(GlobalFilterCollection filters)
            {
                filters.Add(new ErrorHandler.AiHandleErrorAttribute());
            }
        }
    }
    ```

7. Actualice el archivo Web.config de la siguiente manera:

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <!--
      For more information on how to configure your ASP.NET application, please visit
      https://go.microsoft.com/fwlink/?LinkId=301880
      -->
    <configuration>
      <appSettings>
        <add key="webpages:Version" value="3.0.0.0" />
        <add key="webpages:Enabled" value="false" />
        <add key="ClientValidationEnabled" value="true" />
        <add key="UnobtrusiveJavaScriptEnabled" value="true" />
      </appSettings>
      <system.web>
        <compilation debug="true" targetFramework="4.7.2" />
        <httpRuntime targetFramework="4.7.2" />
        <httpModules>
          <add name="TelemetryCorrelationHttpModule" type="Microsoft.AspNet.TelemetryCorrelation.TelemetryCorrelationHttpModule, Microsoft.AspNet.TelemetryCorrelation" />
          <add name="ApplicationInsightsWebTracking" type="Microsoft.ApplicationInsights.Web.ApplicationInsightsHttpModule, Microsoft.AI.Web" />
        </httpModules>
      </system.web>
      <runtime>
        <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
          <dependentAssembly>
            <assemblyIdentity name="Antlr3.Runtime" publicKeyToken="eb42632606e9261f" />
            <bindingRedirect oldVersion="0.0.0.0-3.5.0.2" newVersion="3.5.0.2" />
          </dependentAssembly>
          <dependentAssembly>
            <assemblyIdentity name="Newtonsoft.Json" publicKeyToken="30ad4fe6b2a6aeed" />
            <bindingRedirect oldVersion="0.0.0.0-12.0.0.0" newVersion="12.0.0.0" />
          </dependentAssembly>
          <dependentAssembly>
            <assemblyIdentity name="System.Web.Optimization" publicKeyToken="31bf3856ad364e35" />
            <bindingRedirect oldVersion="1.0.0.0-1.1.0.0" newVersion="1.1.0.0" />
          </dependentAssembly>
          <dependentAssembly>
            <assemblyIdentity name="WebGrease" publicKeyToken="31bf3856ad364e35" />
            <bindingRedirect oldVersion="0.0.0.0-1.6.5135.21930" newVersion="1.6.5135.21930" />
          </dependentAssembly>
          <dependentAssembly>
            <assemblyIdentity name="System.Web.Helpers" publicKeyToken="31bf3856ad364e35" />
            <bindingRedirect oldVersion="1.0.0.0-3.0.0.0" newVersion="3.0.0.0" />
          </dependentAssembly>
          <dependentAssembly>
            <assemblyIdentity name="System.Web.WebPages" publicKeyToken="31bf3856ad364e35" />
            <bindingRedirect oldVersion="1.0.0.0-3.0.0.0" newVersion="3.0.0.0" />
          </dependentAssembly>
          <dependentAssembly>
            <assemblyIdentity name="System.Web.Mvc" publicKeyToken="31bf3856ad364e35" />
            <bindingRedirect oldVersion="1.0.0.0-5.2.7.0" newVersion="5.2.7.0" />
          </dependentAssembly>
          <dependentAssembly>
            <assemblyIdentity name="System.Memory" publicKeyToken="cc7b13ffcd2ddd51" culture="neutral" />
            <bindingRedirect oldVersion="0.0.0.0-4.0.1.1" newVersion="4.0.1.1" />
          </dependentAssembly>
        </assemblyBinding>
      </runtime>
      <system.codedom>
        <compilers>
          <compiler language="c#;cs;csharp" extension=".cs" type="Microsoft.CodeDom.Providers.DotNetCompilerPlatform.CSharpCodeProvider, Microsoft.CodeDom.Providers.DotNetCompilerPlatform, Version=2.0.1.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" warningLevel="4" compilerOptions="/langversion:default /nowarn:1659;1699;1701" />
          <compiler language="vb;vbs;visualbasic;vbscript" extension=".vb" type="Microsoft.CodeDom.Providers.DotNetCompilerPlatform.VBCodeProvider, Microsoft.CodeDom.Providers.DotNetCompilerPlatform, Version=2.0.1.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" warningLevel="4" compilerOptions="/langversion:default /nowarn:41008 /define:_MYTYPE=\&quot;Web\&quot; /optionInfer+" />
        </compilers>
      </system.codedom>
      <system.webServer>
        <validation validateIntegratedModeConfiguration="false" />
        <modules>
          <remove name="TelemetryCorrelationHttpModule" />
          <add name="TelemetryCorrelationHttpModule" type="Microsoft.AspNet.TelemetryCorrelation.TelemetryCorrelationHttpModule, Microsoft.AspNet.TelemetryCorrelation" preCondition="managedHandler" />
          <remove name="ApplicationInsightsWebTracking" />
          <add name="ApplicationInsightsWebTracking" type="Microsoft.ApplicationInsights.Web.ApplicationInsightsHttpModule, Microsoft.AI.Web" preCondition="managedHandler" />
        </modules>
      </system.webServer>
    </configuration>
    
    ```

Ahora ha configurado correctamente la supervisión de aplicaciones del lado servidor. Si ejecuta la aplicación web, podrá ver que la telemetría comienza a aparecer dentro de Application Insights.

## <a name="add-client-side-monitoring"></a>Adición de la supervisión del lado cliente

En las secciones anteriores se proporcionan instrucciones sobre métodos para configurar automática y manualmente la supervisión de servidor. Para agregar la supervisión de cliente, necesitará usar nuestro [SDK de JavaScript del lado cliente](javascript.md). Puede supervisar las transacciones del lado cliente de cualquier página web agregando un [fragmento de código de JavaScript](javascript.md#snippet-based-setup) antes de la etiqueta de cierre `</head>` del código HTML de la página. 

Aunque es posible agregar manualmente el fragmento de código al encabezado de cada página HTML, se recomienda agregarlo a una página principal, lo que inyectará el fragmento de código en todas las páginas de un sitio. Para la aplicación MVC de ASP.NET basada en plantilla de este artículo, el archivo que necesita editar se denomina `_Layout.cshtml` y se encuentra en **Vistas** > **Compartida**.

Para agregar la supervisión de cliente, abra el archivo `_Layout.cshtml` y siga las [instrucciones de instalación basadas en fragmentos de código](javascript.md#snippet-based-setup) del artículo de configuración del SDK de JavaScript del lado cliente.

## <a name="troubleshooting"></a>Solución de problemas

Existe un problema conocido en la versión actual de Visual Studio 2019 que cuando el almacenamiento de la clave de instrumentación en un secreto de usuario está interrumpido para aplicaciones basadas en .NET Framework, en última instancia, la clave tiene que estar codificada en el archivo applicationinsights.config para solucionar este error. Este artículo está diseñado para evitar este problema por completo, para lo cual no se usan secretos de usuario.  

## <a name="open-source-sdk"></a>SDK de código abierto

* [Lectura y contribución al código](https://github.com/microsoft/ApplicationInsights-dotnet).

Para obtener las actualizaciones y correcciones de errores más recientes, [consulte las notas de la versión](./release-notes.md).

## <a name="next-steps"></a>Pasos siguientes

* Agregue transacciones sintéticas para probar que el sitio web está disponible en todo el mundo con [supervisión de disponibilidad](monitor-web-app-availability.md).
* [Configure el muestreo](sampling.md) para ayudar a reducir el tráfico de telemetría y los costos de almacenamiento de datos.


