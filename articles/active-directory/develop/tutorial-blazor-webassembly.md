---
title: 'Tutorial: Inicio de sesión de los usuarios y llamada a una API protegida desde una aplicación WebAssembly de Blazor'
titleSuffix: Microsoft identity platform
description: En este tutorial, iniciará la sesión de los usuarios y llamará a una API protegida mediante la plataforma de identidad de Microsoft en una aplicación Blazor WebAssembly (WASM).
author: knicholasa
ms.author: nichola
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.date: 10/16/2020
ms.openlocfilehash: 7a095cee6b2af93358b0e76c39cfc0c04ad454fa
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111408054"
---
# <a name="tutorial-sign-in-users-and-call-a-protected-api-from-a-blazor-webassembly-app"></a>Tutorial: Inicio de sesión de usuarios y llamada a una API protegida desde una aplicación WebAssembly de Blazor

En este tutorial, creará una aplicación de Blazor WebAssembly que inicia la sesión de usuario y obtiene datos de Microsoft Graph mediante la Plataforma de identidad de Microsoft y el registro de la aplicación en Azure Active Directory (Azure AD). 

En este tutorial:

> [!div class="checklist"]
>
> * Crear una nueva aplicación WebAssembly de Blazor configurada para usar Azure Active Directory (Azure AD) para la [autenticación y autorización](authentication-vs-authorization.md) mediante la plataforma de identidad de Microsoft.
> * Recuperar datos de una API web protegida; en este caso, [Microsoft Graph](/graph/overview).

En este tutorial se usa .NET Core 3.1. Los documentos de .NET contienen instrucciones sobre [cómo proteger una aplicación Blazor WebAssembly](/aspnet/core/blazor/security/webassembly/graph-api) mediante ASP.NET Core 5.0. 

También hay disponible un [tutorial para Blazor Server](tutorial-blazor-server.md). 

## <a name="prerequisites"></a>Requisitos previos

* [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1)
* Un inquilino de Azure AD donde pueda registrar una aplicación. Si no tiene acceso a un inquilino de Azure AD, puede obtener uno registrándose en el [programa para desarrolladores de Microsoft 365](https://developer.microsoft.com/microsoft-365/dev-program) o creando una [cuenta gratuita de Azure](https://azure.microsoft.com/free).

## <a name="register-the-app-in-the-azure-portal"></a>Registro de la aplicación en Azure Portal

Todas las aplicaciones que utilizan Azure Active Directory (Azure AD) para la autenticación deben registrarse con Azure AD. Siga las instrucciones de [Registro de una aplicación](quickstart-register-app.md) con estas especificaciones:

- Para la opción **Tipos de cuenta admitidos**, seleccione **Solo las cuentas de este directorio organizativo**.
- En la lista desplegable **URI de redirección**, seleccione **Aplicación de página única (SPA)** y escriba `https://localhost:5001/authentication/login-callback`. El puerto predeterminado de una aplicación que se ejecuta en Kestrel es 5001. Si la aplicación está disponible en un puerto diferente, especifique el número de puerto en lugar de `5001`.

Una vez que se haya registrado, en **Administrar**, seleccione **Autenticación** > **Implicit grant and hybrid flows** (Concesión implícita y flujos híbridos). Seleccione **Tokens de acceso** y **Tokens de id.** , y haga clic en **Guardar**.

## <a name="create-the-app-using-the-net-core-cli"></a>Creación de la aplicación mediante la CLI de .NET Core

Para crear la aplicación, necesita las plantillas de Blazor más recientes. Puede instalarlas para la CLI de .NET Core con el siguiente comando:

```dotnetcli
dotnet new -i Microsoft.Identity.Web.ProjectTemplates::1.6.0
```

Después, ejecute el siguiente comando para crear la aplicación. Reemplace los marcadores de posición del comando por la información adecuada de la página de información general de la aplicación y ejecute el comando en un shell de comandos. La ubicación de salida especificada con la opción `-o|--output` crea una carpeta de proyecto si no existe y se convierte en parte del nombre de la aplicación.

```dotnetcli
dotnet new blazorwasm2 --auth SingleOrg --calls-graph -o {APP NAME} --client-id "{CLIENT ID}" --tenant-id "{TENANT ID}"
```

| Marcador de posición   | Nombre de Azure Portal       | Ejemplo                                |
| ------------- | ----------------------- | -------------------------------------- |
| `{APP NAME}`  | &mdash;                 | `BlazorWASMSample`                         |
| `{CLIENT ID}` | Id. de aplicación (cliente) | `41451fa7-0000-0000-0000-69eff5a761fd` |
| `{TENANT ID}` | Id. de directorio (inquilino)   | `e86c78e2-0000-0000-0000-918e0565a45e` |

## <a name="test-the-app"></a>Pruebas de la aplicación

Ya puede compilar y ejecutar la aplicación. Al ejecutar esta aplicación de plantilla, debe especificar el marco que se ejecutará con --framework. En este tutorial se usa .NET Standard 2.1, pero la plantilla también admite otros marcos.

```dotnetcli
dotnet run --framework netstandard2.1
```

En el explorador, vaya a `https://localhost:5001` e inicie sesión con una cuenta de usuario de Azure AD para ver la aplicación en ejecución e iniciando la sesión de los usuarios con la plataforma de identidad de Microsoft.

Los componentes de esta plantilla que habilitan los inicios de sesión con Azure AD mediante la plataforma de identidad de Microsoft se explican en este [tema de la documentación de ASP.NET](/aspnet/core/blazor/security/webassembly/standalone-with-azure-active-directory#authentication-package).

## <a name="retrieving-data-from-a-protected-api-microsoft-graph"></a>Recuperación de datos de una API protegida (Microsoft Graph)

[Microsoft Graph](/graph/overview) contiene API que proporcionan acceso a los datos de Microsoft 365 para sus usuarios y admite los tokens emitidos por la plataforma de identidad de Microsoft, lo que la convierte en una API protegida adecuada que se puede utilizar como ejemplo. En esta sección, agregará código para llamar a Microsoft Graph y mostrar los mensajes de correo electrónico del usuario en la página de "captura de datos" de la aplicación.

Esta sección se ha redactado con el mismo enfoque que una llamada a una API protegida mediante un cliente con nombre. Se puede utilizar el mismo método para otras API protegidas a las que desee llamar. Sin embargo, si tiene previsto llamar a Microsoft Graph desde la aplicación, puede usar el SDK de Graph para reducir el texto reutilizable. Los documentos de .NET contienen instrucciones sobre [cómo utilizar el SDK de Graph](/aspnet/core/blazor/security/webassembly/graph-api?view=aspnetcore-5.0&preserve-view=true).

Antes de empezar, cierre la sesión de la aplicación, ya que realizará cambios en los permisos necesarios y el token actual no funcionará. Si aún no lo ha hecho, vuelva a ejecutar la aplicación y seleccione **Cerrar sesión** antes de actualizar el código siguiente.

Ahora se actualizará el registro y el código de la aplicación para extraer los correos electrónicos de un usuario y mostrar los mensajes en la aplicación.

En primer lugar, agregue el permiso de API `Mail.Read` al registro de la aplicación para que Azure AD sea consciente de que la aplicación solicitará acceso al correo electrónico de sus usuarios.

1. En Azure Portal, seleccione la aplicación en **Registros de aplicaciones**.
1. En **Administrar**, seleccione **Permisos de API**.
1. Seleccione **Agregar un permiso** > **Microsoft Graph**.
1. Seleccione **Permisos delegados** y, a continuación, busque y seleccione el permiso **Mail.Read**.
1. Seleccione **Agregar permisos**.

A continuación, agregue la siguiente línea al archivo *.csproj* del proyecto dentro de **ItemGroup** netstandard2.1. Esto le permitirá crear el HttpClient personalizado en el paso siguiente.

```xml
<PackageReference Include="Microsoft.Extensions.Http" Version="3.1.7" />
```

A continuación, modifique el código como se indica en los pasos siguientes. Estos cambios agregarán [tokens de acceso](access-tokens.md) a las solicitudes salientes enviadas a Microsoft Graph API. Este patrón se describe con más detalle en [Otros escenarios de seguridad de Blazor WebAssembly en ASP.NET Core](/aspnet/core/blazor/security/webassembly/additional-scenarios).

En primer lugar, cree un nuevo archivo denominado *GraphAPIAuthorizationMessageHandler.cs* con el código siguiente. Este controlador se usará para agregar un token de acceso para los ámbitos `User.Read` y `Mail.Read` a las solicitudes salientes a Microsoft Graph API.

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class GraphAPIAuthorizationMessageHandler : AuthorizationMessageHandler
{
    public GraphAPIAuthorizationMessageHandler(IAccessTokenProvider provider,
        NavigationManager navigationManager)
        : base(provider, navigationManager)
    {
        ConfigureHandler(
            authorizedUrls: new[] { "https://graph.microsoft.com" },
            scopes: new[] { "https://graph.microsoft.com/User.Read", "https://graph.microsoft.com/Mail.Read" });
    }
}
```

Después, reemplace el contenido del método `Main` de *Program.cs* por el código siguiente. Este código hace uso del nuevo controlador `GraphAPIAuthorizationMessageHandler` y agrega `User.Read` y `Mail.Read` como ámbitos predeterminados que la aplicación solicitará cuando el usuario inicie sesión por primera vez.

```csharp
var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("app");

builder.Services.AddScoped<GraphAPIAuthorizationMessageHandler>();

builder.Services.AddHttpClient("GraphAPI",
        client => client.BaseAddress = new Uri("https://graph.microsoft.com"))
    .AddHttpMessageHandler<GraphAPIAuthorizationMessageHandler>();

builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
    options.ProviderOptions.DefaultAccessTokenScopes.Add("User.Read");
    options.ProviderOptions.DefaultAccessTokenScopes.Add("Mail.Read");
});

await builder.Build().RunAsync();
```

Por último, reemplace el contenido de la página *FetchData.Razor* por el código siguiente. Este código captura los datos de correo electrónico del usuario de Microsoft Graph API y los muestra como una lista. En `OnInitializedAsync`, se crea el nuevo `HttpClient` que usa el token de acceso adecuado y se utiliza para hacer la solicitud a Microsoft Graph API.

```c#
@page "/fetchdata"
@using System.ComponentModel.DataAnnotations
@using System.Text.Json.Serialization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@using Microsoft.Extensions.Logging
@inject IAccessTokenProvider TokenProvider
@inject IHttpClientFactory ClientFactory
@inject IHttpClientFactory HttpClientFactory

<p>This component demonstrates fetching data from a service.</p>

@if (messages == null)
{
    <p><em>Loading...</em></p>
}
else
{
    <h1>Hello @userDisplayName !!!!</h1>
    <table class="table">
        <thead>
            <tr>
                <th>Subject</th>
                <th>Sender</th>
                <th>Received Time</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var mail in messages)
            {
                <tr>
                    <td>@mail.Subject</td>
                    <td>@mail.Sender</td>
                    <td>@mail.ReceivedTime</td>
                </tr>
            }
        </tbody>
    </table>
}

@code {

    private string userDisplayName;
    private List<MailMessage> messages = new List<MailMessage>();

    private HttpClient _httpClient;

    protected override async Task OnInitializedAsync()
    {
        _httpClient = HttpClientFactory.CreateClient("GraphAPI");
        try {
            var dataRequest = await _httpClient.GetAsync("https://graph.microsoft.com/beta/me");

            if (dataRequest.IsSuccessStatusCode)
            {
                var userData = System.Text.Json.JsonDocument.Parse(await dataRequest.Content.ReadAsStreamAsync());
                userDisplayName = userData.RootElement.GetProperty("displayName").GetString();
            }

            var mailRequest = await _httpClient.GetAsync("https://graph.microsoft.com/beta/me/messages?$select=subject,receivedDateTime,sender&$top=10");

            if (mailRequest.IsSuccessStatusCode)
            {
                var mailData = System.Text.Json.JsonDocument.Parse(await mailRequest.Content.ReadAsStreamAsync());
                var messagesArray = mailData.RootElement.GetProperty("value").EnumerateArray();

                foreach (var m in messagesArray)
                {
                    var message = new MailMessage();
                    message.Subject = m.GetProperty("subject").GetString();
                    message.Sender = m.GetProperty("sender").GetProperty("emailAddress").GetProperty("address").GetString();
                    message.ReceivedTime = m.GetProperty("receivedDateTime").GetDateTime();
                    messages.Add(message);
                }
            }
        }
        catch (AccessTokenNotAvailableException ex)
        {
            // Tokens are not valid - redirect the user to log in again
            ex.Redirect();
        }
    }

    public class MailMessage
    {
        public string Subject;
        public string Sender;
        public DateTime ReceivedTime;
    }
}
```

Ahora vuelva a iniciar la aplicación. Observará que se le pedirá que conceda a la aplicación acceso para leer el correo. Esto se espera cuando una aplicación solicita el ámbito `Mail.Read`.

Después de conceder el consentimiento, vaya a la página "Captura de datos" para leer algún correo electrónico.

:::image type="content" source="./media/tutorial-blazor-webassembly/final-app.png" alt-text="Captura de pantalla de la aplicación final. Tiene un encabezado que dice &quot;Hello Nicholas&quot; y muestra una lista de correos electrónicos que pertenecen a Nicholas.":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Procedimientos recomendados y recomendaciones de la plataforma de identidad de Microsoft](./identity-platform-integration-checklist.md)
