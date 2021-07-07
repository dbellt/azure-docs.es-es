---
title: 'Inicio rápido: Incorporación del inicio de sesión con Microsoft Identity a una aplicación web de ASP.NET Core | Azure'
titleSuffix: Microsoft identity platform
description: En este inicio rápido, aprenderá cómo una aplicación implementa el inicio de sesión de Microsoft en una aplicación web de ASP.NET mediante OpenID Connect.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/11/2020
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, identityplatformtop40, scenarios:getting-started, languages:aspnet-core
ms.openlocfilehash: bb58a86aad4e0298f12c917d475520ee44501c00
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110494430"
---
# <a name="quickstart-add-sign-in-with-microsoft-to-an-aspnet-core-web-app"></a>Inicio rápido: Adición del inicio de sesión con Microsoft a una aplicación web de ASP.NET Core

En este inicio rápido, descargará y ejecutará un código de ejemplo que muestra cómo una aplicación web de ASP.NET Core puede realizar el inicio de sesión de los usuarios desde cualquier organización de Azure Active Directory.  

> [!div renderon="docs"]
> El diagrama siguiente muestra cómo funciona la aplicación de ejemplo:
>
> ![Diagrama de la interacción entre el explorador web, la aplicación web y la plataforma de identidad de Microsoft en la aplicación de ejemplo.](media/quickstart-v2-aspnet-core-webapp/aspnetcorewebapp-intro.svg)
>
> ## <a name="prerequisites"></a>Prerrequisitos
>
> * [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) o [Visual Studio Code](https://code.visualstudio.com/)
> * [SDK de .NET Core 3.1+](https://dotnet.microsoft.com/download)
>
> ## <a name="register-and-download-the-app"></a>Registro y descarga de la aplicación
> Tiene dos opciones para empezar a crear la aplicación: usar la configuración automática o la manual.
>
> ### <a name="automatic-configuration"></a>Configuración automática
> Si desea configurar automáticamente la aplicación y, a continuación, descargar el ejemplo de código, siga estos pasos:
>
> 1. Vaya a la <a href="https://aka.ms/aspnetcore2-1-aad-quickstart-v2/" target="_blank">página de Azure Portal para el registro de aplicaciones</a>.
> 1. Escriba un nombre para la aplicación y seleccione **Registrar**.
> 1. Siga las instrucciones para descargar y configurar automáticamente la nueva aplicación con un clic.
>
> ### <a name="manual-configuration"></a>Configuración manual
> Si desea configurar manualmente la aplicación y el ejemplo de código, siga estos pasos.
> #### <a name="step-1-register-your-application"></a>Paso 1: Registrar su aplicación
> 1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>.
> 1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino en el que desea registrar la aplicación.
> 1. Busque y seleccione **Azure Active Directory**.
> 1. En **Administrar**, seleccione **Registros de aplicaciones** >  y, luego, **Nuevo registro**.
> 1. Escriba un **nombre** para la aplicación. Por ejemplo, especifique **AspNetCore-Quickstart**. Los usuarios de la aplicación verá este nombre, que puede cambiar más tarde.
> 1. En **URI de redirección**, escriba **https://localhost:44321/signin-oidc** .
> 1. Seleccione **Registrar**.
> 1. En **Administrar**, seleccione **Autenticación**.
> 1. En **Dirección URL de cierre de sesión del canal frontal**, escriba **https://localhost:44321/signout-oidc** .
> 1. En **Flujos de concesión implícita e híbridos**, seleccione **Tokens de id.**
> 1. Seleccione **Guardar**.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Paso 1: Configuración de la aplicación en Azure Portal
> Para que el código de ejemplo de este inicio rápido funcione:
> - En **URI de redirección**, escriba **https://localhost:44321/** y **https://localhost:44321/signin-oidc** .
> - En **Dirección URL de cierre de sesión del canal frontal**, escriba **https://localhost:44321/signout-oidc** . 
>
> El punto de conexión de autorización emitirá tokens de identificador de solicitud.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Hacer este cambio por mí]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Ya configurada](media/quickstart-v2-aspnet-webapp/green-check.png) La aplicación está configurada con estos atributos.

#### <a name="step-2-download-the-aspnet-core-project"></a>Paso 2: Descarga del proyecto de ASP.NET Core

> [!div renderon="docs"]
> [Descargar la solución de ASP.NET Core](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/aspnetcore3-1.zip)

> [!div renderon="portal" class="sxs-lookup"]
> Ejecute el proyecto.

> [!div renderon="portal" class="sxs-lookup" id="autoupdate" class="nextstepaction"]
> [Descargar el código de ejemplo](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/aspnetcore3-1.zip)

[!INCLUDE [active-directory-develop-path-length-tip](../../../includes/active-directory-develop-path-length-tip.md)]

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Paso 3: La aplicación está configurada y lista para ejecutarse
> Hemos configurado el proyecto con los valores de las propiedades de su aplicación y está preparado para ejecutarse.
> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`
> [!div renderon="docs"]
> #### <a name="step-3-configure-your-aspnet-core-project"></a>Paso 3: Configuración del proyecto de ASP.NET Core
> 1. Extraiga el archivo. zip en una carpeta local cerca de la raíz de la unidad. Por ejemplo, en *C:\Azure-Samples*.
> 
>    Se recomienda extraer el archivo en un directorio próximo a la raíz de la unidad para evitar errores provocados por las limitaciones de longitud de la ruta de acceso en Windows.
> 1. Abra la solución en Visual Studio 2019.
> 1. Abra el archivo *appsettings.json* y modifique el código siguiente:
>
>    ```json
>    "Domain": "Enter the domain of your tenant, e.g. contoso.onmicrosoft.com",
>    "ClientId": "Enter_the_Application_Id_here",
>    "TenantId": "common",
>    ```
>
>    - Reemplace `Enter_the_Application_Id_here` por el identificador de aplicación (cliente) que ha registrado en Azure Portal. Encontrará el valor de **Id. de aplicación (cliente)** en la página **Información general** de la aplicación.
>    - Reemplace `common` por una de las opciones siguientes:
>       - Si la aplicación admite **solo las cuentas de este directorio organizativo**, reemplace este valor por el identificador de directorio (inquilino) (un GUID) o el nombre del inquilino (por ejemplo, `contoso.onmicrosoft.com`). Encontrará el valor de **Id. de directorio (inquilino)** en la página **Información general** de la aplicación.
>       - Si la aplicación admite **Cuentas en cualquier directorio organizativo**, reemplace este valor por `organizations`.
>       - Si la aplicación admite **Todos los usuarios de cuentas Microsoft**, establezca el valor en `common`.
>
> Para esta guía de inicio rápido, no cambie ningún otro valor del archivo *appsettings.json*.
>
> #### <a name="step-4-build-and-run-the-application"></a>Paso 4: Compilación y ejecución de la aplicación
>
> Compile y ejecute la aplicación en Visual Studio. Para ello, seleccione el menú **Depurar** > **Iniciar depuración** o presione la tecla F5.
>
> Se le solicitarán las credenciales y, a continuación, que acepte los permisos que requiere la aplicación. Seleccione **Aceptar** en la solicitud de consentimiento.
>
> :::image type="content" source="media/quickstart-v2-aspnet-core-webapp/webapp-01-consent.png" alt-text="Captura de pantalla del cuadro de diálogo de consentimiento, que muestra los permisos que la aplicación solicita al usuario.":::
>
> Después de que haya aceptado los permisos solicitados, la aplicación muestra que ha iniciado sesión correctamente con sus credenciales de Azure Active Directory.
>
> :::image type="content" source="media/quickstart-v2-aspnet-core-webapp/webapp-02-signed-in.png" alt-text="Captura de pantalla que muestra un navegar web donde se ve la aplicación web en ejecución y el usuario que ha iniciado sesión.":::

## <a name="more-information"></a>Más información

En esta sección, se proporciona una introducción al código necesario para el inicio de sesión de usuarios. Esta introducción puede ser útil para comprender cómo funciona el código, cuáles son los argumentos principales y cómo agregar el inicio de sesión a una aplicación ASP.NET Core existente.

> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="how-the-sample-works"></a>Funcionamiento del ejemplo
>
> ![Diagrama de la interacción entre el explorador web, la aplicación web y la plataforma de identidad de Microsoft en la aplicación de ejemplo.](media/quickstart-v2-aspnet-core-webapp/aspnetcorewebapp-intro.svg)

### <a name="startup-class"></a>Clase Startup

El middleware *Microsoft.AspNetCore.Authentication* usa una clase `Startup` que se ejecuta al iniciarse el proceso de hospedaje:

```csharp
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
          .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"));

      services.AddControllersWithViews(options =>
      {
          var policy = new AuthorizationPolicyBuilder()
              .RequireAuthenticatedUser()
              .Build();
          options.Filters.Add(new AuthorizeFilter(policy));
      });
      services.AddRazorPages()
          .AddMicrosoftIdentityUI();
  }
```

El método `AddAuthentication()` configura el servicio para agregar la autenticación basada en cookies. Esta autenticación se usa en escenarios de explorador y para establecer el desafío en OpenID Connect.

La línea que contiene `.AddMicrosoftIdentityWebApp` agrega la autenticación de la Plataforma de identidad de Microsoft a la aplicación. Posteriormente, la aplicación se configura para iniciar sesión mediante la plataforma de identidad de Microsoft de acuerdo con la información de la sección `AzureAD` del archivo de configuración *appsettings.json*:

| Clave de *appsettings.json* | Descripción                                                                                                                                                          |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ClientId`             | El Id. de aplicación (cliente) de la aplicación registrada en Azure Portal.                                                                                       |
| `Instance`             | El punto de conexión del servicio de token de seguridad (STS) para que el usuario se autentique. Normalmente, este valor es `https://login.microsoftonline.com/`, que indica la nube pública de Azure. |
| `TenantId`             | Nombre o identificador (GUID) del inquilino, o bien `common`, para el inicio de sesión de usuarios con cuentas profesionales o educativas o cuentas personales de Microsoft.                             |

El método `Configure()` contiene dos métodos importantes, `app.UseAuthentication()` y `app.UseAuthorization()`, que habilitan su funcionalidad con nombre. También en el método `Configure()`, debe registrar las rutas de Microsoft Identity Web mediante al menos una llamada a `endpoints.MapControllerRoute()` o a `endpoints.MapControllers()`:

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

### <a name="attribute-for-protecting-a-controller-or-methods"></a>Atributo para proteger el controlador o los métodos

Puede proteger un controlador o los métodos de controlador mediante el atributo `[Authorize]`. Este atributo restringe el acceso al controlador o los métodos al permitir únicamente usuarios autenticados. Si el usuario aún no se ha autenticado, se puede iniciar un desafío de autenticación para acceder al controlador.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Pasos siguientes

El repositorio de GitHub que contiene este tutorial de ASP.NET Core incluye instrucciones y más ejemplos de código que muestran cómo:

- Agregar la autenticación a una nueva aplicación web de ASP.NET Core.
- Llamar a Microsoft Graph, otras API de Microsoft o sus propias API web.
- Agregar la autorización.
- Iniciar la sesión de usuarios en nubes nacionales o con identidades sociales.

> [!div class="nextstepaction"]
> [Tutoriales de aplicaciones web de ASP.NET Core en GitHub](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/)
