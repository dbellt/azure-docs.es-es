---
title: Inicio rápido de Azure App Configuration con Azure Functions | Microsoft Docs
description: En este inicio rápido, creará una aplicación de Azure Functions con Azure App Configuration y C#. Cree y conéctese a un almacén de App Configuration. Pruebe la función en un entorno local.
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.custom: devx-track-csharp
ms.topic: quickstart
ms.date: 06/02/2021
ms.author: alkemper
ms.openlocfilehash: cf76cdc52ba7b4ebcf99e0b32d9b56e59a9bf837
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111411672"
---
# <a name="quickstart-create-an-azure-functions-app-with-azure-app-configuration"></a>Inicio rápido: Creación de una aplicación de Azure Functions con Azure App Configuration

En este inicio rápido se incorpora el servicio Azure App Configuration en una aplicación de Azure Functions para centralizar el almacenamiento y la administración de toda la configuración de la aplicación de forma independiente del código.

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/dotnet)
- [Visual Studio 2019](https://visualstudio.microsoft.com/vs) con la carga de trabajo de **desarrollo de Azure**.
- [Herramientas de Azure Functions](../azure-functions/functions-develop-vs.md#check-your-tools-version)

## <a name="create-an-app-configuration-store"></a>Creación de un almacén de App Configuration

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

7. Seleccione **Explorador de configuración** >  **+ Crear** > **Clave-valor** para agregar los siguientes pares clave-valor:

    | Clave | Value |
    |---|---|
    | TestApp:Settings:Message | Datos de Azure App Configuration |

    Deje **Etiqueta** y **Tipo de contenido** en blanco, por ahora.

8. Seleccione **Aplicar**.

## <a name="create-a-functions-app"></a>Creación de una aplicación de Functions

[!INCLUDE [Create a project using the Azure Functions template](../../includes/functions-vstools-create.md)]

## <a name="connect-to-an-app-configuration-store"></a>Conexión a un almacén de App Configuration
En este proyecto se usa la [inserción de dependencias en Azure Functions para .NET](../azure-functions/functions-dotnet-dependency-injection.md) y se agrega Azure App Configuration como origen de configuración adicional.

1. Haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet**. En la pestaña **Examinar**, busque los siguientes paquetes NuGet y agréguelos al proyecto.
   - [Microsoft.Extensions.Configuration.AzureAppConfiguration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureAppConfiguration/), versión 4.1.0 o posterior
   - [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions/), versión 1.1.0 o posterior 

2. Agregue un nuevo archivo, *Startup.cs*, con el código siguiente. Se define una clase denominada `Startup` que implementa la clase abstracta `FunctionsStartup`. Para especificar el nombre de tipo empleado durante el inicio de Azure Functions, se usa un atributo de ensamblado.

    El método `ConfigureAppConfiguration` se invalida y se llama a `AddAzureAppConfiguration()` para agregar el proveedor de Azure App Configuration como origen de configuración adicional. El método `Configure` se deja vacío, ya que no es necesario registrar ningún servicio en este momento.
    
    ```csharp
    using System;
    using Microsoft.Azure.Functions.Extensions.DependencyInjection;
    using Microsoft.Extensions.Configuration;

    [assembly: FunctionsStartup(typeof(FunctionApp.Startup))]

    namespace FunctionApp
    {
        class Startup : FunctionsStartup
        {
            public override void ConfigureAppConfiguration(IFunctionsConfigurationBuilder builder)
            {
                string cs = Environment.GetEnvironmentVariable("ConnectionString");
                builder.ConfigurationBuilder.AddAzureAppConfiguration(cs);
            }

            public override void Configure(IFunctionsHostBuilder builder)
            {
            }
        }
    }
    ```

3. Abra *Function1.cs* y agregue el siguiente espacio de nombres.

    ```csharp
    using Microsoft.Extensions.Configuration;
    ```

   Agregue un constructor utilizado para obtener instancias de `IConfiguration` mediante la inserción de dependencias.

    ```csharp
    private readonly IConfiguration _configuration;

    public Function1(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    ```

4. Actualice el método `Run` para que lea los valores de la configuración.

    ```csharp
    public async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req, ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        string keyName = "TestApp:Settings:Message";
        string message = _configuration[keyName];

        return message != null
            ? (ActionResult)new OkObjectResult(message)
            : new BadRequestObjectResult($"Please create a key-value with the key '{keyName}' in App Configuration.");
    }
    ```

    > [!NOTE]
    > La clase `Function1` y el método `Run` no deben ser estáticos. Quite el modificador `static` si se ha generado automáticamente.

## <a name="test-the-function-locally"></a>Prueba local de la función

1. Establezca una variable de entorno llamada **ConnectionString** y defínala como la clave de acceso a su almacén de App Configuration. Si usa el símbolo del sistema de Windows, ejecute el siguiente comando y reinícielo para que se aplique el cambio:

    ```cmd
        setx ConnectionString "connection-string-of-your-app-configuration-store"
    ```

    Si usa Windows PowerShell, ejecute el siguiente comando:

    ```azurepowershell
        $Env:ConnectionString = "connection-string-of-your-app-configuration-store"
    ```

    Si usa macOS o Linux, ejecute el siguiente comando:

    ```bash
        export ConnectionString='connection-string-of-your-app-configuration-store'
    ```

2. Presione F5 para probar la función. Si se le solicita, acepte la solicitud de Visual Studio para descargar e instalar las herramientas de **Azure Functions Core (CLI)** . También es preciso que habilite una excepción de firewall para que las herramientas para controlen las solicitudes de HTTP.

3. Copie la dirección URL de la función de los resultados del runtime de Azure Functions.

    ![Inicio rápido: depuración de funciones en VS](./media/quickstarts/function-visual-studio-debugging.png)

4. Pegue la dirección URL de la solicitud HTTP en la barra de direcciones del explorador. La siguiente imagen muestra la respuesta en el explorador para la solicitud GET local devuelta por la función.

    ![Inicio rápido: inicio de funciones locales](./media/quickstarts/dotnet-core-function-launch-local.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de App Configuration y lo ha usado con una aplicación de Azure Functions a través del [proveedor de App Configuration](/dotnet/api/Microsoft.Extensions.Configuration.AzureAppConfiguration). Para aprender a actualizar una aplicación de Azure Functions para poner al día dinámicamente la configuración, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Habilitación de la configuración dinámica en Azure Functions](./enable-dynamic-configuration-azure-functions-csharp.md)
