---
title: 'Tutorial: Uso de la configuración dinámica de App Configuration en ASP.NET Core'
titleSuffix: Azure App Configuration
description: En este tutorial aprenderá a actualizar dinámicamente los datos de configuración de aplicaciones de ASP.NET Core
services: azure-app-configuration
documentationcenter: ''
author: AlexandraKemperMS
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 09/1/2020
ms.author: alkemper
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: c6a80a4d17fd5bf9584a6aaa8b50802f5a4ec5a6
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110468890"
---
# <a name="tutorial-use-dynamic-configuration-in-an-aspnet-core-app"></a>Tutorial: Uso de la configuración dinámica en una aplicación de ASP.NET Core

ASP.NET Core dispone de un sistema de configuración conectable que puede leer datos de configuración de diversos orígenes. Es capaz de controlar los cambios de forma dinámica sin hacer que una aplicación se reinicie. ASP.NET Core admite el enlace de valores de configuración a clases .NET fuertemente tipadas. Las inserta en el código mediante `IOptionsSnapshot<T>`, que vuelve a cargar automáticamente la configuración de la aplicación cuando cambian los datos subyacentes.

Este tutorial le muestra cómo puede implementar las actualizaciones de configuración dinámica en el código. Se basa en la aplicación web que se introdujo en los inicios rápidos. Antes de continuar, finalice primero el tutorial [Creación de una aplicación ASP.NET Core con Azure App Configuration](./quickstart-aspnet-core-app.md).

Para realizar los pasos de este tutorial, puede usar cualquier editor de código. [Visual Studio Code](https://code.visualstudio.com/) es una excelente opción que está disponible en las plataformas Windows, macOS y Linux.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar la aplicación para actualizar su configuración en respuesta a los cambios en un almacén de App Configuration.
> * Insertar la configuración más reciente en los controladores de la aplicación

## <a name="prerequisites"></a>Prerrequisitos

Para realizar este tutorial, instale el [SDK de .NET Core](https://dotnet.microsoft.com/download).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Antes de continuar, finalice primero el tutorial [Creación de una aplicación ASP.NET Core con Azure App Configuration](./quickstart-aspnet-core-app.md).

## <a name="add-a-sentinel-key"></a>Adición de una clave de Sentinel

Una *clave de Sentinel* es una clave especial que se actualiza después de completar el cambio de todas las demás claves. La aplicación supervisa la clave de Sentinel. Cuando se detecta un cambio, la aplicación actualiza todos los valores de configuración. Este enfoque ayuda a garantizar la coherencia de la configuración en la aplicación y reduce el número total de solicitudes realizadas a App Configuration, en comparación con la supervisión de los cambios en todas las claves.

1. En Azure Portal, seleccione **Explorador de configuración > Crear > Clave-valor**.
1. En **Clave**, escriba *TestApp:Settings:Sentinel*. En **Valor**, escriba 1. Deje **Etiqueta** y **Tipo de contenido** en blanco.
1. Seleccione **Aplicar**.

> [!NOTE]
> Si no utiliza una clave de Sentinel, deberá registrar manualmente cada clave que quiera ver.

## <a name="reload-data-from-app-configuration"></a>Recarga de datos de App Configuration

1. Para agregar una referencia al paquete NuGet `Microsoft.Azure.AppConfiguration.AspNetCore`, ejecute el comando siguiente:

    ```dotnetcli
    dotnet add package Microsoft.Azure.AppConfiguration.AspNetCore
    ```

1. Abra *Program.cs* y actualice el método `CreateWebHostBuilder` para agregar el método `config.AddAzureAppConfiguration()`.

   #### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
                webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
                {
                    var settings = config.Build();
                    config.AddAzureAppConfiguration(options =>
                    {
                        options.Connect(settings["ConnectionStrings:AppConfig"])
                               .ConfigureRefresh(refresh =>
                                    {
                                        refresh.Register("TestApp:Settings:Sentinel", refreshAll: true)
                                               .SetCacheExpiration(new TimeSpan(0, 5, 0));
                                    });
                    });
                })
            .UseStartup<Startup>());
    ```   
    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
                webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
                {
                    var settings = config.Build();
                    config.AddAzureAppConfiguration(options =>
                    {
                        options.Connect(settings["ConnectionStrings:AppConfig"])
                               .ConfigureRefresh(refresh =>
                                    {
                                        refresh.Register("TestApp:Settings:Sentinel", refreshAll: true)
                                               .SetCacheExpiration(new TimeSpan(0, 5, 0));
                                    });
                    });
                })
            .UseStartup<Startup>());
    ```
    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();

                config.AddAzureAppConfiguration(options =>
                {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                           .ConfigureRefresh(refresh =>
                                {
                                    refresh.Register("TestApp:Settings:Sentinel", refreshAll: true)
                                           .SetCacheExpiration(new TimeSpan(0, 5, 0));
                                });
                });
            })
            .UseStartup<Startup>();
    ```
    ---

    En el método `ConfigureRefresh`, puede registrar las claves en el almacén de App Configuration que desea supervisar para detectar los cambios. El parámetro `refreshAll` del método `Register` indica que se deben actualizar todos los valores de configuración si cambia la clave registrada. El método `SetCacheExpiration` especifica el tiempo mínimo que debe transcurrir antes de que se realice una nueva solicitud a App Configuration para comprobar si hay cambios de configuración. En este ejemplo, se reemplaza el tiempo de expiración predeterminado de 30 segundos y especifica un tiempo de 5 minutos en su lugar. Esto reduce el número posible de solicitudes realizadas al almacén de App Configuration.

    > [!NOTE]
    > Puede que desee reducir el tiempo de expiración de la actualización de la memoria caché con fines de prueba.

    Para desencadenar realmente una actualización de la configuración, usará el middleware de App Configuration. Puede ver cómo hacerlo en un paso posterior.

1. Agregue el archivo *Settings.cs* al directorio Controllers que define e implementa una nueva clase `Settings`. Reemplace el espacio de nombres por el nombre del proyecto. 

    ```csharp
    namespace TestAppConfig
    {
        public class Settings
        {
            public string BackgroundColor { get; set; }
            public long FontSize { get; set; }
            public string FontColor { get; set; }
            public string Message { get; set; }
        }
    }
    ```

1. Abra *Startup.cs* y actualice el método `ConfigureServices`. Llame a `Configure<Settings>` para enlazar datos de configuración a la clase `Settings`. Llame a `AddAzureAppConfiguration` para agregar los componentes de App Configuration a la colección de servicios de la aplicación.

    #### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<Settings>(Configuration.GetSection("TestApp:Settings"));
        services.AddControllersWithViews();
        services.AddAzureAppConfiguration();
    }
    ```
    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<Settings>(Configuration.GetSection("TestApp:Settings"));
        services.AddControllersWithViews();
        services.AddAzureAppConfiguration();
    }
    ```
    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<Settings>(Configuration.GetSection("TestApp:Settings"));
        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
        services.AddAzureAppConfiguration();
    }
    ```
    ---

1. Actualice el método `Configure` y agregue una llamada a `UseAzureAppConfiguration`. Esto permite a la aplicación usar el middleware de App Configuration para controlar automáticamente las actualizaciones de configuración.

    #### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                app.UseHsts();
            }

            // Add the following line:
            app.UseAzureAppConfiguration();

            app.UseHttpsRedirection();
            
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Home}/{action=Index}/{id?}");
            });
    }
    ```
    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                app.UseHsts();
            }

            // Add the following line:
            app.UseAzureAppConfiguration();

            app.UseHttpsRedirection();
            
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Home}/{action=Index}/{id?}");
            });
    }
    ```
    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseAzureAppConfiguration();

        services.Configure<CookiePolicyOptions>(options =>
        {
            options.CheckConsentNeeded = context => true;
            options.MinimumSameSitePolicy = SameSiteMode.None;
        });

        app.UseMvc();
    }
    ```
    ---
    
    > [!NOTE]
    > El middleware de App Configuration supervisa la clave de Sentinel o cualquier otra clave que haya registrado para la actualización en la llamada a `ConfigureRefresh` del paso anterior. El middleware se desencadena en cada solicitud entrante a la aplicación. Sin embargo, el middleware solo enviará solicitudes para comprobar el valor de App Configuration cuando haya transcurrido el tiempo de expiración de la caché establecido. Cuando se detecta un cambio, se actualizará toda la configuración, si se usa la clave de Sentinel, o solo se actualizarán los valores de las claves registradas.
    > - Si se produce un error en una solicitud de detección de cambios de App Configuration, la aplicación seguirá utilizando la configuración almacenada en la caché. Se realizará otra comprobación cuando el tiempo de expiración de la caché configurado haya transcurrido de nuevo y haya nuevas solicitudes entrantes a la aplicación.
    > - La actualización de la configuración se produce de forma asincrónica con respecto al procesamiento de las solicitudes entrantes de la aplicación. No bloqueará ni ralentizará la solicitud entrante que desencadenó la actualización. Es posible que no se actualicen los valores de configuración de la solicitud que desencadenó la actualización, pero sí lo harán las solicitudes posteriores.
    > - Para asegurarse de que el middleware se desencadena, llame a `app.UseAzureAppConfiguration()` tan pronto como sea adecuado en la canalización de solicitudes para que otro middleware no provoque un cortocircuito en la aplicación.

## <a name="use-the-latest-configuration-data"></a>Uso de los datos de configuración más recientes

1. Abra *HomeController.cs* en el directorio de controladores y agregue una referencia al paquete `Microsoft.Extensions.Options`.

    ```csharp
    using Microsoft.Extensions.Options;
    ```

2. Actualice la clase `HomeController` para recibir `Settings` mediante la inserción de dependencias y asegúrese de usar sus valores.

    #### <a name="net-5x"></a>[.NET 5.x](#tab/core5x)

    ```csharp
    public class HomeController : Controller
    {
        private readonly Settings _settings;
        private readonly ILogger<HomeController> _logger;

        public HomeController(ILogger<HomeController> logger, IOptionsSnapshot<Settings> settings)
        {
            _logger = logger;
            _settings = settings.Value;
        }

        public IActionResult Index()
        {
            ViewData["BackgroundColor"] = _settings.BackgroundColor;
            ViewData["FontSize"] = _settings.FontSize;
            ViewData["FontColor"] = _settings.FontColor;
            ViewData["Message"] = _settings.Message;

            return View();
        }

        // ...
    }
    ```
    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public class HomeController : Controller
    {
        private readonly Settings _settings;
        private readonly ILogger<HomeController> _logger;

        public HomeController(ILogger<HomeController> logger, IOptionsSnapshot<Settings> settings)
        {
            _logger = logger;
            _settings = settings.Value;
        }

        public IActionResult Index()
        {
            ViewData["BackgroundColor"] = _settings.BackgroundColor;
            ViewData["FontSize"] = _settings.FontSize;
            ViewData["FontColor"] = _settings.FontColor;
            ViewData["Message"] = _settings.Message;

            return View();
        }

        // ...
    }
    ```
    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public class HomeController : Controller
    {
        private readonly Settings _settings;
        public HomeController(IOptionsSnapshot<Settings> settings)
        {
            _settings = settings.Value;
        }

        public IActionResult Index()
        {
            ViewData["BackgroundColor"] = _settings.BackgroundColor;
            ViewData["FontSize"] = _settings.FontSize;
            ViewData["FontColor"] = _settings.FontColor;
            ViewData["Message"] = _settings.Message;

            return View();
        }
    }
    ```
    ---
    > [!Tip]
    > Para obtener más información sobre el patrón de opciones al leer valores de configuración, consulte [Patrón de opciones en ASP.NET Core](/aspnet/core/fundamentals/configuration/options).

3. Abra el archivo *Index.cshtml* en el directorio Views > Home, y sustituya su contenido por el siguiente script:

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <style>
        body {
            background-color: @ViewData["BackgroundColor"]
        }
        h1 {
            color: @ViewData["FontColor"];
            font-size: @ViewData["FontSize"]px;
        }
    </style>
    <head>
        <title>Index View</title>
    </head>
    <body>
        <h1>@ViewData["Message"]</h1>
    </body>
    </html>
    ```

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Para compilar la aplicación mediante la CLI de .NET Core, ejecute el siguiente comando en el shell de comandos:

    ```console
        dotnet build
    ```

1. Una vez que la compilación se haya realizado correctamente, ejecute el siguiente comando para ejecutar la aplicación web localmente:

    ```console
        dotnet run
    ```

1. Abra una ventana del explorador y vaya a la dirección URL que aparece en la salida `dotnet run`.

    ![Inicio local de la aplicación de inicio rápido](./media/quickstarts/aspnet-core-app-launch-local-before.png)

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Todos los recursos** y seleccione la instancia de almacén de App Configuration que creó en el inicio rápido.

1. Seleccione **Explorador de configuración** y actualice los valores de las claves siguientes. No olvide actualizar la clave de Sentinel al final.

    | Clave | Value |
    |---|---|
    | TestApp:Settings:BackgroundColor | green |
    | TestApp:Settings:FontColor | lightGray |
    | TestApp:Settings:Message | Datos de Azure App Configuration: ahora con actualizaciones directas |
    | TestApp:Settings:Sentinel | 2 |

1. Actualice la página del explorador para ver los nuevos valores de configuración. Es posible que necesite actualizar el explorador más de una vez para que se reflejen los cambios o que tenga que cambiar el tiempo de expiración de la caché para que sea inferior a 5 minutos. 

    ![Inicio local de la aplicación de inicio rápido actualizada](./media/quickstarts/aspnet-core-app-launch-local-after.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha habilitado la aplicación web de ASP.NET Core para actualizar dinámicamente la configuración a partir de App Configuration. Para aprender a usar una identidad administrada de Azure para simplificar el acceso a App Configuration, vaya al siguiente tutorial.

> [!div class="nextstepaction"]
> [Integración de identidades administradas](./howto-integrate-azure-managed-service-identity.md)
