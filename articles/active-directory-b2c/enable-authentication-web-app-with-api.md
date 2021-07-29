---
title: Habilitación de la autenticación en una web que llama a una API web mediante bloques de creación de Azure Active Directory B2C
description: Los bloques de creación de una aplicación web de ASP.NET que llama a una API web mediante Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 06/11/2021
ms.author: mimart
ms.subservice: B2C
ms.custom: b2c-support
ms.openlocfilehash: 53c81633508bef928faf8919c618b9ad30dd3844
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112073192"
---
# <a name="enable-authentication-in-your-own-web-application-that-calls-a-web-api-using-azure-active-directory-b2c"></a>Habilitación de la autenticación en su propia aplicación web que llama a una API web mediante Azure Active Directory B2C

En este artículo se muestra cómo agregar la autenticación de Azure Active Directory B2C (Azure AD B2C) a su propia aplicación web de ASP.NET que llama a una API web. Aprenda a crear una aplicación web de ASP.NET Core con middleware de ASP.NET Core que usa el protocolo [OpenID Connect](openid-connect.md). Use este artículo junto con el artículo [Configuración de la autenticación en una aplicación web de ejemplo que llama a una API web](configure-authentication-sample-web-app-with-api.md) cuando sustituya la aplicación web de ejemplo por la suya propia.

Este artículo se centra en el proyecto de aplicación web. Para obtener instrucciones sobre cómo crear la API web, consulte el [ejemplo de la API web de la lista de tareas](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/4-WebApp-your-API/4-2-B2C).

## <a name="prerequisites"></a>Prerrequisitos

Revise los requisitos previos y los pasos de integración de [Configuración de la autenticación en una aplicación web de ejemplo que llama a una API web](configure-authentication-sample-web-app-with-api.md).

## <a name="create-a-web-app-project"></a>Creación de un proyecto de aplicación web

Puede usar un proyecto de aplicación web MVC de ASP.NET existente o crear uno nuevo. Para crear un nuevo proyecto, abra un shell de comandos y escriba el siguiente comando:

```dotnetcli
dotnet new mvc -o mywebapp
```

El comando anterior:

* Crea una nueva aplicación web MVC.  
* El parámetro `-o mywebapp` crea un directorio llamado *mywebapp* con los archivos de código fuente de la aplicación.

## <a name="add-the-authentication-libraries"></a>Adición de las bibliotecas de autenticación

En primer lugar, agregue la biblioteca web de identidad de Microsoft. Se trata de un conjunto de bibliotecas de ASP.NET Core que simplifican la adición de la compatibilidad con la autenticación y la autorización de Azure AD B2C en la aplicación web. La biblioteca Microsoft Identity Web configura la canalización de autenticación con la autenticación basada en cookies. Se encarga de enviar y recibir mensajes de autenticación HTTP, la validación de tokens, la extracción de notificaciones, etc.

Para agregar la biblioteca Microsoft Identity Web, instale los paquetes mediante la ejecución de los siguientes comandos: 

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```dotnetcli
dotnet add package Microsoft.Identity.Web
dotnet add package Microsoft.Identity.Web.UI
```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```dotnetcli
Install-Package Microsoft.Identity.Web
Install-Package Microsoft.Identity.Web.UI
```

---


## <a name="initiate-the-authentication-libraries"></a>Inicio de las bibliotecas de autenticación

El middleware Microsoft Identity Web usa una clase de inicio que se ejecuta cuando se inicia el proceso de hospedaje. En este paso, agregará el código necesario para iniciar las bibliotecas de autenticación.

Abra `Startup.cs` y agregue las siguientes declaraciones `using` al principio de la clase :

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.Identity.Web;
using Microsoft.Identity.Web.UI;
```

Dado que Microsoft Identity Web usa la autenticación basada en cookies para proteger la aplicación web, el código siguiente establece la configuración de cookies *SameSite*. A continuación, lee la configuración de la aplicación `AzureADB2C` e inicia el controlador de middleware con su vista. 

Reemplace la función `ConfigureServices(IServiceCollection services)` por el fragmento de código siguiente. 

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<CookiePolicyOptions>(options =>
    {
        // This lambda determines whether user consent for non-essential cookies is needed for a given request.
        options.CheckConsentNeeded = context => true;
        options.MinimumSameSitePolicy = SameSiteMode.Unspecified;
        // Handling SameSite cookie according to https://docs.microsoft.com/en-us/aspnet/core/security/samesite?view=aspnetcore-3.1
        options.HandleSameSiteCookieCompatibility();
    });

    // Configuration to sign-in users with Azure AD B2C
    services.AddMicrosoftIdentityWebAppAuthentication(Configuration, "AzureAdB2C")
            // Enable token acquisition to call downstream web API
            .EnableTokenAcquisitionToCallDownstreamApi(new string[] { Configuration["TodoList:TodoListScope"] })
            // Add refresh token in-memory cache
            .AddInMemoryTokenCaches();

    services.AddControllersWithViews()
        .AddMicrosoftIdentityUI();

    services.AddRazorPages();

    //Configuring appsettings section AzureAdB2C, into IOptions
    services.AddOptions();
    services.Configure<OpenIdConnectOptions>(Configuration.GetSection("AzureAdB2C"));
}
```

El código siguiente agrega la directiva de cookies y usa el modelo de autenticación. Reemplace la función `Configure` por el fragmento de código siguiente. 

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

    app.UseHttpsRedirection();
    app.UseStaticFiles();

    // Add the Microsoft Identity Web cookie policy
    app.UseCookiePolicy();
    app.UseRouting();
    // Add the ASP.NET Core authentication service
    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller=Home}/{action=Index}/{id?}");
        
        // Add endpoints for Razor pages
        endpoints.MapRazorPages();
    });
};
```

## <a name="add-the-ui-elements"></a>Adición de elementos de la interfaz de usuario

Para agregar elementos de la interfaz de usuario, use una vista parcial. La vista parcial contiene lógica para comprobar si un usuario ha iniciado sesión o no. Si el usuario no ha iniciado sesión, la vista parcial representa el botón de inicio de sesión. Si el usuario ha iniciado sesión, se muestra el nombre para mostrar del usuario y el botón de cerrar sesión.
  
Cree un nuevo archivo `_LoginPartial.cshtml` dentro de la carpeta `Views/Shared` con el siguiente fragmento de código:

```razor
@using System.Security.Principal
@if (User.Identity.IsAuthenticated)
{
    <ul class="nav navbar-nav navbar-right">
        <li class="navbar-text">Hello @User.Identity.Name</li>
        <!-- The Account controller is not defined in this project. Instead, it is part of Microsoft.Identity.Web.UI nuget package and
            it defines some well known actions such as SignUp/In, SignOut and EditProfile-->
        <li class="navbar-btn">
            <form method="get" asp-area="MicrosoftIdentity" asp-controller="Account" asp-action="EditProfile">
                <button type="submit" class="btn btn-primary" style="margin-right:5px">Edit Profile</button>
            </form>
        </li>
        <li class="navbar-btn">
            <form method="get" asp-area="MicrosoftIdentity" asp-controller="Account" asp-action="SignOut">
                <button type="submit" class="btn btn-primary">Sign Out</button>
            </form>
        </li>
    </ul>
}
else
{
    <ul class="nav navbar-nav navbar-right">
        <li class="navbar-btn">
            <form method="get" asp-area="MicrosoftIdentity" asp-controller="Account" asp-action="SignIn">
                <button type="submit" class="btn btn-primary">Sign Up/In</button>
            </form>
        </li>
    </ul>
}
```

Modifique `Views\Shared\_Layout.cshtml` para incluir el archivo *_LoginPartial.cshtml* que agregó. El archivo *_Layout.cshtml* es un diseño común que proporciona al usuario una experiencia coherente a medida que navega de página a página. El diseño incluye elementos comunes de la interfaz de usuario, como el encabezado y el pie de página de la aplicación.

> [!NOTE]
> Dependiendo de la versión de .NET Core y de si va a agregar el inicio de sesión a una aplicación existente, los elementos de la interfaz de usuario podrían tener un aspecto diferente. Si es así, asegúrese de incluir *_LoginPartial* en la ubicación adecuada dentro del diseño de página.

Abra */Views/Shared/_Layout.cshtml* y agregue el siguiente elemento `div`.

```razor
<div class="navbar-collapse collapse">
...
</div>
```

Reemplace este elemento con el siguiente código de Razor:

```razor
<div class="navbar-collapse collapse">
  <ul class="nav navbar-nav">
    <li><a asp-area="" asp-controller="Home" asp-action="Index">Home</a></li>
    <li><a asp-area="" asp-controller="Home" asp-action="Claims">Claims</a></li>
    <li><a asp-area="" asp-controller="Home" asp-action="TodoList">To do list</a></li>
  </ul>
  <partial name="_LoginPartial" />
</div>
```

El código de Razor anterior incluye un vínculo a las acciones `Claims` y `TodoList` que creará en los siguientes pasos.

## <a name="add-the-claims-view"></a>Adición de la vista de notificaciones

Para ver las notificaciones del token de identificador en la carpeta `Views/Home`, agregue la vista `Claims.cshtml`.

```razor
@using System.Security.Claims

@{
  ViewData["Title"] = "Claims";
}
<h2>@ViewData["Title"].</h2>

<table class="table-hover table-condensed table-striped">
  <tr>
    <th>Claim Type</th>
    <th>Claim Value</th>
  </tr>

  @foreach (Claim claim in User.Claims)
  {
    <tr>
      <td>@claim.Type</td>
      <td>@claim.Value</td>
    </tr>
  }
</table>
```

En este paso, agregará la acción `Claims` que vincula la vista *Claims.cshtml* al controlador *Home*. Usa el atributo `[Authorize]`, que limita el acceso a la acción Claims a los usuarios autenticados.  

En el controlador `/Controllers/HomeController.cs`, agregue la siguiente acción.

```csharp
[Authorize]
public IActionResult Claims()
{
    return View();
}
```

Agregue la siguiente declaración `using` al comienzo de la clase:

```csharp
using Microsoft.AspNetCore.Authorization;
```

## <a name="add-the-to-do-list-view"></a>Adición de la vista de lista de tareas pendientes

Para llamar a la API web de tareas pendientes, debe tener un token de acceso con los ámbitos adecuados. En este paso, accederá y actuará en el controlador `Home`. En la carpeta `Views/Home`, agregue la vista `TodoList.cshtml`.

```razor
@{
    ViewData["Title"] = "To do list";
}

<div class="text-left">
  <h1 class="display-4">Your access token</h1>
  @* Remove following line in production environments *@
  <code>@ViewData["accessToken"]</code>
</div>
```

Después de agregar la vista, agregue la acción `TodoList` que vincula la vista *TodoList.cshtml* al controlador *Home*. Usa el atributo `[Authorize]`, que limita el acceso a la acción TodoList a los usuarios autenticados.  

En el controlador `/Controllers/HomeController.cs`, agregue el siguiente miembro de clase de acción e inyecte el servicio de adquisición de tokens en el controlador.

```csharp
public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;

    // Add the token acquisition service member variable
    private readonly ITokenAcquisition _tokenAcquisition; 
    
    // Inject the acquisition service
    public HomeController(ILogger<HomeController> logger, ITokenAcquisition tokenAcquisition)
    {
        _logger = logger;
        // Set the acquisition service member variable
        _tokenAcquisition = tokenAcquisition;
    }

    // More code...
}
```

Después, agregue la acción siguiente. La acción muestra cómo llamar a una API web junto con el token de portador. 

```csharp
[Authorize]
public async Task<IActionResult> TodoListAsync()
{
    // Acquire an access token with the relevant scopes.
    var accessToken = await _tokenAcquisition.GetAccessTokenForUserAsync(new[] { "https://your-tenant.onmicrosoft.com/tasks-api/tasks.read", "https://your-tenant.onmicrosoft.com/tasks-api/tasks.write" });
    
    // Remove this line in production environments    
    ViewData["accessToken"] = accessToken;

    using (HttpClient client = new HttpClient())
    {
        client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

        HttpResponseMessage response = await client.GetAsync("https://path-to-your-web-api");
    }

    return View();
}
```

## <a name="add-the-app-settings"></a>Adición de la configuración de la aplicación

La configuración del proveedor de identidades de Azure AD B2C se almacena en el archivo `appsettings.json`. Abra appsettings.json y agregue la configuración de la aplicación como se describe en [Paso 5: Configuración de la aplicación web de ejemplo](configure-authentication-sample-web-app-with-api.md#step-5-configure-the-sample-web-app).

## <a name="run-your-application"></a>Ejecución de la aplicación

1. Compile y ejecute el proyecto.
1. Vaya a https://localhost:5001. 
1. Seleccione **SignIn/Up** (Iniciar sesión o registrarse).
1. Complete el proceso de inicio de sesión o registro.

Después de autenticarse correctamente, verá el nombre para mostrar en la barra de navegación. 

* Para ver las notificaciones que devuelve el token de Azure AD B2C a la aplicación, seleccione **Notificaciones**.
* Para ver el token de acceso, seleccione **Lista de tareas pendientes**.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [personalizar y mejorar la experiencia de autenticación de Azure AD B2C para la aplicación web](enable-authentication-web-application-options.md).
