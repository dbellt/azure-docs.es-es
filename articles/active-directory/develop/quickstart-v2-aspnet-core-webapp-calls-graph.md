---
title: 'Inicio rápido: aplicación web ASP.NET Core que inicia la sesión de los usuarios y llama a Microsoft Graph | Azure'
titleSuffix: Microsoft identity platform
description: En este inicio rápido, obtendrá información sobre la forma en que una aplicación usa Microsoft.Identity.Web para implementar el inicio de sesión de Microsoft en una aplicación web ASP.NET Core mediante OpenID Connect y llama a Microsoft Graph
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 05/17/2021
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, scenarios:getting-started, languages:aspnet-core
ms.openlocfilehash: e93e3619ebf63d27d1739da75e1f5762aad4b502
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110086040"
---
# <a name="quickstart-aspnet-core-web-app-that-signs-in-users-and-calls-microsoft-graph-on-their-behalf"></a>Inicio rápido: aplicación web ASP.NET Core que inicia la sesión de los usuarios y llama a Microsoft Graph en su nombre

En este inicio rápido, descargará y ejecutará un código de ejemplo que muestra la forma en que una aplicación web de ASP.NET Core puede realizar el inicio de sesión de los usuarios desde cualquier organización de Azure Active Directory y llama a Microsoft Graph.

Para ilustrar este tema, consulte el apartado en el que se explica el [funcionamiento del ejemplo](#how-the-sample-works).

> [!div renderon="docs"]
> ## <a name="prerequisites"></a>Prerrequisitos
>
> * [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) o [Visual Studio Code](https://code.visualstudio.com/)
> * [SDK de .NET Core 3.1+](https://dotnet.microsoft.com/download)
>
>
> ## <a name="step-1-register-your-application"></a>Paso 1: Registrar su aplicación
> Para registrar la aplicación y agregar la información de registro de la aplicación a la solución de forma manual, siga estos pasos:
>
> 1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
> 1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino en el que desea registrar una aplicación.
> 1. Busque y seleccione **Azure Active Directory**.
> 1. En **Administrar**, seleccione **Registros de aplicaciones** >  y, luego, **Nuevo registro**.
> 1. Escriba el **Nombre** de la aplicación, por ejemplo `AspNetCoreWebAppCallsGraph-Quickstart`. Los usuarios de la aplicación pueden ver este nombre, el cual se puede cambiar más tarde.
> 1. Escriba un **URI de redirección** de `https://localhost:44321/signin-oidc`.
> 1. Seleccione **Registrar**.
> 1. En **Administrar**, seleccione **Autenticación**.
> 1. Especifique un valor para **Front-channel logout URL** (Dirección URL de cierre de sesión del canal frontal) de `https://localhost:44321/signout-oidc`.
> 1. Seleccione **Guardar**.
> 1. En **Administrar**, seleccione **Certificados y secretos** > **Nuevo secreto de cliente**.
> 1. Escriba un valor en **Descripción**, por ejemplo`clientsecret1`.
> 1. Seleccione **En 1 año** como expiración del secreto.
> 1. Seleccione **Agregar** y registre inmediatamente el **valor** del secreto para usarlo en un paso posterior. Este valor de secreto *no se volverá a mostrar* y no se puede recuperar de ninguna otra manera. Regístrelo en una ubicación segura, tal como haría con cualquier contraseña.

> [!div class="sxs-lookup" renderon="portal"]
> ## <a name="step-1-configure-your-application-in-the-azure-portal"></a>Paso 1: Configuración de la aplicación en Azure Portal
>
> Para que el código de ejemplo de este inicio rápido funcione, agregue un **URI de redirección**  de `https://localhost:44321/signin-oidc` y un valor para **Front-channel logout URL** (Dirección URL de cierre de sesión del canal frontal) de `https://localhost:44321/signout-oidc` en el registro de la aplicación.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Hacer este cambio por mí]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Ya configurada](media/quickstart-v2-aspnet-webapp/green-check.png) La aplicación está configurada con estos atributos.

## <a name="step-2-download-the-aspnet-core-project"></a>Paso 2: Descarga del proyecto de ASP.NET Core

> [!div renderon="docs"]
> [Descargar la solución de ASP.NET Core](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/aspnetcore3-1-callsgraph.zip)

> [!div renderon="portal" class="sxs-lookup"]
> Ejecute el proyecto.

> [!div renderon="portal" class="sxs-lookup" id="autoupdate" class="nextstepaction"]
> [Descargar el código de ejemplo](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/aspnetcore3-1-callsgraph.zip)

[!INCLUDE [active-directory-develop-path-length-tip](../../../includes/active-directory-develop-path-length-tip.md)]

> [!div class="sxs-lookup" renderon="portal"]
> ## <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Paso 3: La aplicación está configurada y lista para ejecutarse
>
> Hemos configurado el proyecto con los valores de las propiedades de su aplicación y está preparado para ejecutarse.
> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`
> [!div renderon="docs"]
>
> ## <a name="step-3-configure-your-aspnet-core-project"></a>Paso 3: Configuración del proyecto de ASP.NET Core
> 1. Extraiga el archivo. zip en una carpeta local cerca de la raíz de la unidad. Por ejemplo, en *C:\Azure-Samples*.
> 1. Abra la solución en Visual Studio 2019.
> 1. Abra el archivo *appsettings.json* y modifique lo siguiente:
>
>    ```json
>    "ClientId": "Enter_the_Application_Id_here",
>    "TenantId": "common",
>    "clientSecret": "Enter_the_Client_Secret_Here"
>    ```
>
>    - Reemplace `Enter_the_Application_Id_here` por el **Id. de aplicación (cliente)** de la aplicación que ha registrado en Azure Portal. Puede encontrar el **Identificador de aplicación (cliente)** en la página **Información general** de la aplicación.
>    - Reemplace `common` por una de las opciones siguientes:
>       - Si la aplicación admite **Solo las cuentas de este directorio organizativo**, reemplace este valor por el **Id. de directorio (inquilino)** (un GUID) o por el **Nombre del inquilino** (por ejemplo, `contoso.onmicrosoft.com`). Puede encontrar el **Id. de directorio (inquilino)** en la página **Introducción** de la aplicación.
>       - Si la aplicación admite **Cuentas en cualquier directorio organizativo**, reemplace este valor por `organizations`
>       - Si la aplicación admite **Todos los usuarios de cuentas Microsoft**, deje este valor establecido en `common`.
>    - Reemplace `Enter_the_Client_Secret_Here` por el **secreto de cliente** que creó y registró en un paso anterior.
>
> Para esta guía de inicio rápido, no cambie ningún otro valor del archivo *appsettings.json*.
>
> ## <a name="step-4-build-and-run-the-application"></a>Paso 4: Compilación y ejecución de la aplicación
>
> Para compilar y ejecutar la aplicación en Visual Studio, seleccione el menú **Depurar** > **Iniciar depuración** o presione la tecla `F5`.
>
> Se le pedirán las credenciales y, a continuación, se le pedirá que dé su consentimiento a los permisos que requiere la aplicación. Seleccione **Aceptar** en la solicitud de consentimiento.
>
> :::image type="content" source="media/quickstart-v2-aspnet-core-webapp-calls-graph/webapp-01-consent.png" alt-text="Cuadro de diálogo de consentimiento que muestra los permisos que solicita la aplicación al usuario >":::
>
> Tras el consentimiento a los permisos solicitados, la aplicación muestra que ha iniciado sesión correctamente con sus credenciales de Azure Active Directory y verá su dirección de correo electrónico en la sección "Api result" (Resultado de la API) de la página. La extracción se realizó mediante Microsoft Graph.
>
> :::image type="content" source="media/quickstart-v2-aspnet-core-webapp-calls-graph/webapp-02-signed-in.png" alt-text="Explorador web que muestra la aplicación web en ejecución y el usuario con la sesión iniciada":::

## <a name="about-the-code"></a>Sobre el código

En esta sección se proporciona información general sobre el código necesario para iniciar sesión a los usuarios y llamar a Microsoft Graph API en su nombre. Esta descripción puede ser útil para conocer el funcionamiento del código, los argumentos principales y si se desea agregar el inicio de sesión a una aplicación ASP.NET Core existente y agregar Microsoft Graph. Utiliza [Microsoft.Identity.Web](microsoft-identity-web.md), que es un contenedor de [MSAL.NET](msal-overview.md).

### <a name="how-the-sample-works"></a>Funcionamiento del ejemplo

![Muestra cómo funciona la aplicación de ejemplo generada por este inicio rápido.](media/quickstart-v2-aspnet-core-webapp-calls-graph/aspnetcorewebapp-intro.svg)

### <a name="startup-class"></a>Clase Startup

El middleware *Microsoft.AspNetCore.Authentication* usa una clase `Startup` que se ejecuta cuando se inicializa el proceso de hospedaje:

```csharp
  // Get the scopes from the configuration (appsettings.json)
  var initialScopes = Configuration.GetValue<string>("DownstreamApi:Scopes")?.Split(' ');

  public void ConfigureServices(IServiceCollection services)
  {
      // Add sign-in with Microsoft
      services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
        .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"))

            // Add the possibility of acquiring a token to call a protected web API
            .EnableTokenAcquisitionToCallDownstreamApi(initialScopes)

                // Enables controllers and pages to get GraphServiceClient by dependency injection
                // And use an in memory token cache
                .AddMicrosoftGraph(Configuration.GetSection("DownstreamApi"))
                .AddInMemoryTokenCaches();

      services.AddControllersWithViews(options =>
      {
          var policy = new AuthorizationPolicyBuilder()
              .RequireAuthenticatedUser()
              .Build();
          options.Filters.Add(new AuthorizeFilter(policy));
      });

      // Enables a UI and controller for sign in and sign out.
      services.AddRazorPages()
          .AddMicrosoftIdentityUI();
  }
```

El método `AddAuthentication()` configura el servicio para agregar la autenticación basada en cookies, que se usa en escenarios con explorador, y para establecer el desafío en OpenId Connect.

La línea que contiene `.AddMicrosoftIdentityWebApp` agrega la autenticación de la Plataforma de identidad de Microsoft a la aplicación. La proporciona [Microsoft.Identity.Web](microsoft-identity-web.md). Después se configura para iniciar sesión mediante la plataforma de identidad de Microsoft según la información de la sección `AzureAD` del archivo de configuración *appsettings.json*:

| Clave de *appsettings.json* | Descripción                                                                                                                                                          |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ClientId`             | El **Id. de aplicación (cliente)** de la aplicación registrada en Azure Portal.                                                                                       |
| `Instance`             | El punto de conexión del servicio de token de seguridad (STS) para que el usuario se autentique. Normalmente, este valor es `https://login.microsoftonline.com/`, que indica la nube pública de Azure. |
| `TenantId`             | Nombre o id. (un GUID) del inquilino, o bien *common*, para el inicio de sesión de usuarios con cuentas profesionales o educativas o cuentas personales de Microsoft.                             |

El método `EnableTokenAcquisitionToCallDownstreamApi` permite a la aplicación adquirir un token para llamar a las API web protegidas. `AddMicrosoftGraph` permite a los controladores o páginas de Razor beneficiarse directamente de `GraphServiceClient` (mediante la inserción de dependencias) y el método `AddInMemoryTokenCaches` permite a la aplicación beneficiarse de un caché de token.

El método `Configure()` contiene dos métodos importantes, `app.UseAuthentication()` y `app.UseAuthorization()`, que habilitan su funcionalidad con nombre. También en el método `Configure()`, debe registrar las rutas de Microsoft Identity Web con al menos una llamada a `endpoints.MapControllerRoute()` o a `endpoints.MapControllers()`.

```csharp
app.UseAuthentication();
app.UseAuthorization();

app.UseEndpoints(endpoints =>
{

    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
    endpoints.MapRazorPages();
});

// endpoints.MapControllers(); // REQUIRED if MapControllerRoute() isn't called.
```

### <a name="protect-a-controller-or-a-controllers-method"></a>Protección de un controlador o del método de un controlador

Para proteger un controlador o sus métodos, aplique el atributo `[Authorize]` a la clase del controlador o a uno o varios de sus métodos. Este atributo `[Authorize]` restringe el acceso al controlador, ya que solo permite usuarios autenticados. Si el usuario aún no está autenticado, se puede iniciar un desafío de autenticación para acceder al controlador. En este inicio rápido, los ámbitos se leen desde el archivo de configuración:

```csharp
[AuthorizeForScopes(ScopeKeySection = "DownstreamApi:Scopes")]
public async Task<IActionResult> Index()
{
    var user = await _graphServiceClient.Me.Request().GetAsync();
    ViewData["ApiResult"] = user.DisplayName;

    return View();
}
```

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Pasos siguientes

El repositorio de GitHub que contiene el ejemplo de código de ASP.NET Core al que se hace referencia en este inicio rápido incluye instrucciones y más ejemplos de código que muestran cómo:

- Agregar autenticación a una nueva aplicación web de ASP.NET Core
- Llamar a Microsoft Graph u otras API de Microsoft en sus propias API web
- Agregar autorización
- Iniciar la sesión de usuarios en nubes nacionales o con identidades de redes sociales

> [!div class="nextstepaction"]
> [Tutoriales de aplicaciones web de ASP.NET Core en GitHub](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/)
