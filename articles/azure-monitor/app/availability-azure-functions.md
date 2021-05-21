---
title: Creación y ejecución de pruebas de disponibilidad personalizadas mediante Azure Functions
description: En este documento se explica cómo crear una función de Azure con TrackAvailability() que se ejecutará periódicamente según la configuración especificada en la función TimerTrigger. Los resultados de esta prueba se enviarán al recurso de Application Insights, donde podrá consultar los datos de los resultados de disponibilidad y generar alertas sobre ellos. Las pruebas personalizadas le permitirán escribir pruebas de disponibilidad más complejas de las que puede crear con la interfaz de usuario del portal, supervisar una aplicación dentro de la red virtual de Azure, cambiar la dirección del punto de conexión o crear una prueba de disponibilidad si no hay ninguna disponible en su región.
ms.topic: conceptual
ms.date: 05/06/2021
ms.openlocfilehash: 891f69bd42dc228e01e7b341630e7f37522f9828
ms.sourcegitcommit: c1b0d0b61ef7635d008954a0d247a2c94c1a876f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109628194"
---
# <a name="create-and-run-custom-availability-tests-using-azure-functions"></a>Creación y ejecución de pruebas de disponibilidad personalizadas mediante Azure Functions

En este artículo se explica cómo crear una función de Azure con TrackAvailability() que se ejecutará periódicamente según la configuración especificada en la función TimerTrigger con lógica de negocios propia. Los resultados de esta prueba se enviarán al recurso de Application Insights, donde podrá consultar los datos de los resultados de disponibilidad y generar alertas sobre ellos. Esto le permite crear pruebas personalizadas de forma parecida a lo que puede hacer mediante [Supervisión de la disponibilidad](./monitor-web-app-availability.md) en el portal. Las pruebas personalizadas le permitirán escribir pruebas de disponibilidad más complejas de las que puede crear con la interfaz de usuario del portal, supervisar una aplicación dentro de la red virtual de Azure, cambiar la dirección del punto de conexión o crear una prueba de disponibilidad incluso si esta característica no está disponible en su región.

> [!NOTE]
> Este ejemplo se ha diseñado únicamente para mostrar la mecánica de cómo actúa la llamada a la API de TrackAvailability() en una función de Azure. No muestra cómo se escribe el código de prueba HTTP o la lógica de negocios subyacentes que se necesitarían para convertir esta llamada en una prueba de disponibilidad totalmente funcional. De forma predeterminada, si sigue este ejemplo, creará una prueba HTTP GET de disponibilidad básica.

## <a name="create-a-timer-trigger-function"></a>Creación de una función de desencadenador de temporizador

1. Cree un recurso de Azure Functions personalizado.
    - Si ya tiene un recurso de Application Insights:
        - De forma predeterminada Azure Functions crea un recurso de Application Insights, pero si desea usar uno de los recursos que ya ha creado, deberá especificarlo durante la creación.
        - Siga las instrucciones sobre cómo [crear un recurso de Azure Functions](../../azure-functions/functions-create-scheduled-function.md#create-a-function-app) con la modificación siguiente:
            - En la pestaña **Supervisión**, seleccione el cuadro desplegable Application Insights y escriba o seleccione el nombre del recurso.
                :::image type="content" source="media/availability-azure-functions/app-insights-resource.png" alt-text="En la pestaña Supervisión, seleccione el recurso de Application Insights existente.":::
    - Si aún no tiene un recurso de Application Insights creado para la función desencadenada por el temporizador:
        - De forma predeterminada, si crea una aplicación de Azure Functions, se creará un recurso de Application Insights. Siga las instrucciones sobre cómo [crear un recurso de Azure Functions](../../azure-functions/functions-create-scheduled-function.md#create-a-function-app).
    > [!NOTE]
    > Puede hospedar las funciones en un plan de consumo, Premium o App Service. Si realiza pruebas detrás de una red virtual o prueba puntos de conexión no públicos, deberá usar el plan Premium en lugar del de consumo. Seleccione el plan en la pestaña **Hospedaje**.
2. Cree una función de desencadenador de temporizador.
    1. En su aplicación de función, seleccione la pestaña **Funciones**.
    1. Seleccione **Agregar** y, en la pestaña Agregar función, seleccione las siguientes configuraciones:
        1. Entorno de desarrollo, seleccione *Desarrollar en el portal*.
        1. Seleccione una plantilla: *Desencadenador de temporizador*
    1. Seleccione **Agregar** para crear la función Desencadenador de temporizador.

    :::image type="content" source="media/availability-azure-functions/add-function.png" alt-text="Captura de pantalla de cómo agregar una función de desencadenador de temporizador a la aplicación de funciones." lightbox="media/availability-azure-functions/add-function.png":::

## <a name="add-and-edit-code-in-the-app-service-editor"></a>Adición y edición de código en el editor de App Service

Vaya a la aplicación de funciones implementada y, en *Herramientas de desarrollo,* seleccione la pestaña **Editor de App Service**.

Para crear un nuevo archivo, haga clic con el botón derecho en la función de desencadenador de temporizador (por ejemplo, "TimerTrigger1") y seleccione **Nuevo archivo**. A continuación, escriba el nombre del archivo y presione Entrar.

1. Cree un archivo nuevo denominado "function.proj" y pegue en él el código siguiente:

    ```xml
    <Project Sdk="Microsoft.NET.Sdk"> 
        <PropertyGroup> 
            <TargetFramework>netstandard2.0</TargetFramework> 
        </PropertyGroup> 
        <ItemGroup> 
            <PackageReference Include="Microsoft.ApplicationInsights" Version="2.15.0" /> <!-- Ensure you’re using the latest version --> 
        </ItemGroup> 
    </Project> 
    
    ```

     :::image type="content" source="media/availability-azure-functions/function-proj.png" alt-text="Captura de pantalla de function.proj en el Editor de App Service." lightbox="media/availability-azure-functions/function-proj.png":::

2. Cree un archivo nuevo denominado "runAvailabilityTest.csx" y pegue el código siguiente:

    ```csharp
    using System.Net.Http; 
    
    public async static Task RunAvailabilityTestAsync(ILogger log) 
    { 
        using (var httpClient = new HttpClient()) 
        { 
            // TODO: Replace with your business logic 
            await httpClient.GetStringAsync("https://www.bing.com/"); 
        } 
    } 
    ```

3. Copie el código siguiente en el archivo run.csx (esto reemplazará el código ya existente):

    ```csharp
    #load "runAvailabilityTest.csx" 
    
    using System; 
    
    using System.Diagnostics; 
    
    using Microsoft.ApplicationInsights; 
    
    using Microsoft.ApplicationInsights.Channel; 
    
    using Microsoft.ApplicationInsights.DataContracts; 
    
    using Microsoft.ApplicationInsights.Extensibility; 
    
    private static TelemetryClient telemetryClient; 
    
    // ============================================================= 
    
    // ****************** DO NOT MODIFY THIS FILE ****************** 
    
    // Business logic must be implemented in RunAvailabilityTestAsync function in runAvailabilityTest.csx 
    
    // If this file does not exist, please add it first 
    
    // ============================================================= 
    
    public async static Task Run(TimerInfo myTimer, ILogger log, ExecutionContext executionContext) 
    
    { 
        if (telemetryClient == null) 
        { 
            // Initializing a telemetry configuration for Application Insights based on connection string 
    
            var telemetryConfiguration = new TelemetryConfiguration(); 
            telemetryConfiguration.ConnectionString = Environment.GetEnvironmentVariable("APPLICATIONINSIGHTS_CONNECTION_STRING"); 
            telemetryConfiguration.TelemetryChannel = new InMemoryChannel(); 
            telemetryClient = new TelemetryClient(telemetryConfiguration); 
        } 
    
        string testName = executionContext.FunctionName; 
        string location = Environment.GetEnvironmentVariable("REGION_NAME"); 
        var availability = new AvailabilityTelemetry 
        { 
            Name = testName, 
    
            RunLocation = location, 
    
            Success = false, 
        }; 
    
        availability.Context.Operation.ParentId = Activity.Current.SpanId.ToString(); 
        availability.Context.Operation.Id = Activity.Current.RootId; 
        var stopwatch = new Stopwatch(); 
        stopwatch.Start(); 
    
        try 
        { 
            using (var activity = new Activity("AvailabilityContext")) 
            { 
                activity.Start(); 
                availability.Id = Activity.Current.SpanId.ToString(); 
                // Run business logic 
                await RunAvailabilityTestAsync(log); 
            } 
            availability.Success = true; 
        } 
    
        catch (Exception ex) 
        { 
            availability.Message = ex.Message; 
            throw; 
        } 
    
        finally 
        { 
            stopwatch.Stop(); 
            availability.Duration = stopwatch.Elapsed; 
            availability.Timestamp = DateTimeOffset.UtcNow; 
            telemetryClient.TrackAvailability(availability); 
            telemetryClient.Flush(); 
        } 
    } 
    
    ```

## <a name="check-availability"></a>Comprobación de la disponibilidad

Para asegurarse de que todo funciona, puede examinar el gráfico de la pestaña Disponibilidad del recurso de Application Insights.

> [!NOTE]
> Las pruebas creadas con TrackAvailability() aparecerán con **CUSTOM** junto al nombre de la prueba.

 :::image type="content" source="media/availability-azure-functions/availability-custom.png" alt-text="Pestaña Disponibilidad con resultados correctos." lightbox="media/availability-azure-functions/availability-custom.png":::

Para ver los detalles de la transacción completa, seleccione **Correcto** o **Erróneo** en Aumentar detalle y, a continuación, seleccione un ejemplo. También puede obtener los detalles de la transacción de un extremo a otro seleccionando un punto de datos en el gráfico.

:::image type="content" source="media/availability-azure-functions/sample.png" alt-text="Selección de una prueba de disponibilidad de ejemplo.":::

:::image type="content" source="media/availability-azure-functions/end-to-end.png" alt-text="Detalles de la transacción completa." lightbox="media/availability-azure-functions/end-to-end.png":::

## <a name="query-in-logs-analytics"></a>Consulta en Registros (Analytics)

Puede usar registros (Analytics) para ver los resultados de disponibilidad, dependencias, etc. Para más información acerca de los registros, visite [Introducción a las consultas de registros](../logs/log-query-overview.md).

:::image type="content" source="media/availability-azure-functions/availabilityresults.png" alt-text="Resultados de disponibilidad." lightbox="media/availability-azure-functions/availabilityresults.png":::

:::image type="content" source="media/availability-azure-functions/dependencies.png" alt-text="Captura de pantalla que muestra la pestaña Nueva consulta con dependencias limitadas a 50." lightbox="media/availability-azure-functions/dependencies.png":::

## <a name="next-steps"></a>Pasos siguientes

- [Mapa de aplicación](./app-map.md)
- [Diagnósticos de transacción](./transaction-diagnostics.md)
