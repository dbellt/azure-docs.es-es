---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: peiliu
manager: vravikumar
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 06/01/2021
ms.topic: include
ms.custom: include file
ms.author: peiliu
ms.openlocfilehash: fc76538023af79d84becc1662bf2402a3daadff4
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111593136"
---
## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- La versión más reciente del [SDK de .NET Core](https://dotnet.microsoft.com/download/dotnet-core) para su sistema operativo.
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).
- Cree un [recurso de Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/create-new-resource) en Azure Portal.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `TelemetryAppInsightsQuickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: **Program.cs**.

```console
dotnet new console -o TelemetryAppInsightsQuickstart
```

Cambie el directorio a la carpeta de la aplicación recién creada y use el comando `dotnet build` para compilar la aplicación.

```console
cd TelemetryAppInsightsQuickstart
dotnet build
```

### <a name="install-the-package"></a>Instalar el paquete

En el directorio de aplicaciones, instale el paquete de la biblioteca de identidades de Azure Communication Services para .NET usando el comando `dotnet add package`.

```console
dotnet add package Azure.Communication.Identity --version 1.0.0
```

También deberá instalar la biblioteca OpenTelemetry para el exportador de Azure Monitor.

```console
dotnet add package Azure.Monitor.OpenTelemetry.Exporter
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

1. Abra el archivo **Program.cs** en un editor de texto.
2. Agregue una directiva `using` para incluir el espacio de nombres `Azure.Communication.Identity`.
3. Agregue una directiva `using` para incluir el espacio de nombres `Azure.Monitor.OpenTelemetry.Exporter`.
4. Agregue una directiva `using` para incluir el espacio de nombres `OpenTelemetry.Trace`.
5. Actualice la declaración del método `Main` para admitir código asincrónico.

Use el código siguiente para empezar:

```csharp
using System;
using Azure.Communication;
using Azure.Communication.Identity;
using Azure.Monitor.OpenTelemetry.Exporter;
using OpenTelemetry.Trace;

namespace TelemetryAppInsightsQuickstart
{
    class Program
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine("Azure Communication Services - Export Telemetry to Application Insights");

            // Quickstart code goes here
        }
    }
}
```
## <a name="setting-up-the-telemetry-tracer-with-communication-identity-sdk-calls"></a>Configuración del seguimiento de telemetría con llamadas del SDK de identidad de comunicación

Inicialice un objeto `CommunicationIdentityClient` con su cadena de conexión. Aprenda a [administrar la cadena de conexión del recurso](../create-communication-resource.md#store-your-connection-string).

Una vez creado el cliente, debemos definir un `Activity Source` que realice el seguimiento de todas las actividades. A continuación, puede usar `Activity Source` para iniciar un objeto `Activity` que se usará para realizar el seguimiento de la llamada del SDK de `CreateUserAsync`. Tenga en cuenta que también puede definir propiedades personalizadas para realizar su seguimiento dentro de cada `Activity` mediante el método `SetTag`.

El patrón de seguimiento es similar para la función `GetTokenAsync`.

Cree una función denominada `TracedSample` y agregue el código siguiente:

```csharp
public static async Task TracedSample()
{
    // This code demonstrates how to fetch your connection string
    // from an environment variable.
    string connectionString = Environment.GetEnvironmentVariable("COMMUNICATION_SERVICES_CONNECTION_STRING");
    var client = new CommunicationIdentityClient(connectionString);

    using var source = new ActivitySource("Quickstart.IdentityTelemetry");
    CommunicationUserIdentifier identity = null;

    using (var activity = source.StartActivity("Create User Activity"))
    {
        var identityResponse = await client.CreateUserAsync();

        identity = identityResponse.Value;
        Console.WriteLine($"\nCreated an identity with ID: {identity.Id}");
        activity?.SetTag("Identity id", identity.Id);
    }

    using (var activity = source.StartActivity("Get Token Activity"))
    {
        var tokenResponse = await client.GetTokenAsync(identity, scopes: new[] { CommunicationTokenScope.Chat });

        activity?.SetTag("Token value", tokenResponse.Value.Token);
        activity?.SetTag("Expires on", tokenResponse.Value.ExpiresOn);

        Console.WriteLine($"\nIssued an access token with 'chat' scope that expires at {expiresOn}:");
    }
}
```

## <a name="funneling-telemetry-data-to-application-insights"></a>Canalización de los datos de telemetría en Application Insights

Una vez que las llamadas del SDK se hayan encapsulado con objetos Activity, puede agregar el exportador de seguimiento de OpenTelemetry y canalizar los datos en el recurso de Application Insights.

Tiene la opción de definir un diccionario con algunos atributos de recursos que se mostrarán en Application Insights.
A continuación, llame a `AddSource` y use el mismo nombre de Origen de la actividad que se definió en `TracedSample`.
También tendrá que tomar la cadena de conexión del recurso de Application Insights y pasarla a `AddAzureMonitorTraceExporter()`. Esto canalizará los datos de telemetría al recurso de Application Insights.

Por último, llame a y espere a la función `TracedSample()` donde tenemos nuestras llamadas del SDK.

Agregue el código siguiente al método `Main`:

```csharp
var resourceAttributes = new Dictionary<string, object> { { "service.name", "<service-name>" }, { "service.instance.id", "<service-instance-id>" } };
var resourceBuilder = ResourceBuilder.CreateDefault().AddAttributes(resourceAttributes);

using var tracerProvider = Sdk.CreateTracerProviderBuilder()
    .SetResourceBuilder(resourceBuilder)
    .AddSource("Quickstart.IdentityTelemetry")
    .AddAzureMonitorTraceExporter(o =>
    {
        o.ConnectionString = Environment.GetEnvironmentVariable("APPLICATION_INSIGHTS_CONNECTION_STRING");
    })
    .Build();

await TracedSample();
```

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```console
dotnet run
```
