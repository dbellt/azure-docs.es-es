---
title: 'Tutorial: Creación de una aplicación de chat de Blazor Server: Azure SignalR'
description: En este tutorial aprenderá a crear y modificar una aplicación Blazor Server con Azure SignalR Service.
author: JialinXin
ms.service: signalr
ms.topic: tutorial
ms.date: 09/09/2020
ms.author: jixin
ms.openlocfilehash: 5ed5107cadcfbf247b79c18a6a2e391ab3043565
ms.sourcegitcommit: dd425ae91675b7db264288f899cff6add31e9f69
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2021
ms.locfileid: "108331833"
---
# <a name="tutorial-build-a-blazor-server-chat-app"></a>Tutorial: Compilación de una aplicación de chat de Blazor Server

En este tutorial se muestra cómo crear y modificar una aplicación Blazor Server. Aprenderá a:
> [!div class="checklist"] 
> * Crear una sala de chat sencilla con la plantilla de la aplicación Blazor Server.
> * Trabajar con componentes de Razor.
> * Usar el control de eventos y el enlace de datos en los componentes de Razor.
> * Implementar rápidamente en Azure App Service con Visual Studio.
> * Migrar desde SignalR local a Azure SignalR Service.

## <a name="prerequisites"></a>Requisitos previos

* Instalación del [SDK para .NET Core 3.0](https://dotnet.microsoft.com/download/dotnet-core/3.0) (Versión >= 3.0.100)
* Instalación de [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) (Versión >= 16.3)


[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsblazor)

## <a name="build-a-local-chat-room-in-blazor-server-app"></a>Creación de una sala de chat local en una aplicación Blazor Server

A partir de Visual Studio 2019 versión 16.2.0, Azure SignalR Service está integrado en el proceso de publicación de aplicaciones web para que la administración de las dependencias entre la aplicación web y SignalR Service sea mucho más cómoda. Puede trabajar con una instancia de SignalR local en un entorno local de desarrollo y trabajar en Azure SignalR Service para Azure App Service al mismo tiempo sin cambios en el código.

1. Cree una aplicación de chat de Blazor:
   1. En Visual Studio, seleccione **Crear un proyecto**. 
   1. Seleccione **Aplicación Blazor**. 
   1. Asigne un nombre a la aplicación y elija una carpeta. 
   1. Seleccione la plantilla de **Aplicación Blazor Server**.
    
       > [!NOTE]
       > Asegúrese de que ya ha instalado el SDK de .NET Core 3.0 o posterior para que Visual Studio reconozca correctamente la plataforma de destino.
   
       [ ![En Crear un proyecto, seleccione las plantillas de la aplicación Blazor.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-create.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-create.png#lightbox)
   
   5. También puede crear un proyecto ejecutando el comando [`dotnet new`](/dotnet/core/tools/dotnet-new) en la CLI de .NET:
   
       ```dotnetcli
       dotnet new blazorserver -o BlazorChat
       ```
   
1. Agregue un nuevo archivo de C# denominado `BlazorChatSampleHub.cs` y cree una clase `BlazorSampleHub` derivada de la clase `Hub` para la aplicación de chat. Para más información sobre la creación de centros, consulte [Creación y uso de centros de conectividad](/aspnet/core/signalr/hubs#create-and-use-hubs). 
   
   ```cs
   using System;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.SignalR;
   
   namespace BlazorChat
   {
       public class BlazorChatSampleHub : Hub
       {
           public const string HubUrl = "/chat";
   
           public async Task Broadcast(string username, string message)
           {
               await Clients.All.SendAsync("Broadcast", username, message);
           }
   
           public override Task OnConnectedAsync()
           {
               Console.WriteLine($"{Context.ConnectionId} connected");
               return base.OnConnectedAsync();
           }
   
           public override async Task OnDisconnectedAsync(Exception e)
           {
               Console.WriteLine($"Disconnected {e?.Message} {Context.ConnectionId}");
               await base.OnDisconnectedAsync(e);
           }
       }
   }
   ```
   
1. Agregue un punto de conexión para el centro de conectividad del método `Startup.Configure()`.
   
   ```cs
   app.UseEndpoints(endpoints =>
   {
       endpoints.MapBlazorHub();
       endpoints.MapFallbackToPage("/_Host");
       endpoints.MapHub<BlazorChatSampleHub>(BlazorChatSampleHub.HubUrl);
   });
   ```
   
1. Instale el paquete `Microsoft.AspNetCore.SignalR.Client` para usar el cliente de SignalR.

   ```dotnetcli
   dotnet add package Microsoft.AspNetCore.SignalR.Client --version 3.1.7
   ```

1. Cree un nuevo [componente de Razor](/aspnet/core/blazor/components/) llamado `ChatRoom.razor` en la carpeta `Pages` para implementar el cliente de SignalR. Siga los pasos que se indican a continuación o use el archivo [ChatRoom.razor](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/BlazorChat/Pages/ChatRoom.razor).

   1. Agregue la directiva [`@page`](/aspnet/core/mvc/views/razor#page) y las instrucciones "using". Use la directiva [`@inject`](/aspnet/core/mvc/views/razor#inject) para insertar el servicio [`NavigationManager`](/aspnet/core/blazor/fundamentals/routing#uri-and-navigation-state-helpers).
      
      ```razor
      @page "/chatroom"
      @inject NavigationManager navigationManager
      @using Microsoft.AspNetCore.SignalR.Client;
      ```

   1. En la sección `@code`, agregue los siguientes miembros al nuevo cliente de SignalR para enviar y recibir mensajes.
      
      ```razor
      @code {
          // flag to indicate chat status
          private bool _isChatting = false;
          
          // name of the user who will be chatting
          private string _username;
      
          // on-screen message
          private string _message;
          
          // new message input
          private string _newMessage;
          
          // list of messages in chat
          private List<Message> _messages = new List<Message>();
          
          private string _hubUrl;
          private HubConnection _hubConnection;
      
          public async Task Chat()
          {
              // check username is valid
              if (string.IsNullOrWhiteSpace(_username))
              {
                  _message = "Please enter a name";
                  return;
              };
      
              try
              {
                  // Start chatting and force refresh UI.
                  _isChatting = true;
                  await Task.Delay(1);

                  // remove old messages if any
                  _messages.Clear();
         
                  // Create the chat client
                  string baseUrl = navigationManager.BaseUri;
      
                  _hubUrl = baseUrl.TrimEnd('/') + BlazorChatSampleHub.HubUrl;
      
                  _hubConnection = new HubConnectionBuilder()
                      .WithUrl(_hubUrl)
                      .Build();
      
                  _hubConnection.On<string, string>("Broadcast", BroadcastMessage);
      
                  await _hubConnection.StartAsync();
      
                  await SendAsync($"[Notice] {_username} joined chat room.");
              }
              catch (Exception e)
              {
                  _message = $"ERROR: Failed to start chat client: {e.Message}";
                  _isChatting = false;
              }
          }
      
          private void BroadcastMessage(string name, string message)
          {
              bool isMine = name.Equals(_username, StringComparison.OrdinalIgnoreCase);
      
              _messages.Add(new Message(name, message, isMine));
      
              // Inform blazor the UI needs updating
              StateHasChanged();
          }
      
          private async Task DisconnectAsync()
          {
              if (_isChatting)
              {
                  await SendAsync($"[Notice] {_username} left chat room.");
      
                  await _hubConnection.StopAsync();
                  await _hubConnection.DisposeAsync();
      
                  _hubConnection = null;
                  _isChatting = false;
              }
          }
      
          private async Task SendAsync(string message)
          {
              if (_isChatting && !string.IsNullOrWhiteSpace(message))
              {
                  await _hubConnection.SendAsync("Broadcast", _username, message);
      
                  _newMessage = string.Empty;
              }
          }
      
          private class Message
          {
              public Message(string username, string body, bool mine)
              {
                  Username = username;
                  Body = body;
                  Mine = mine;
              }
      
              public string Username { get; set; }
              public string Body { get; set; }
              public bool Mine { get; set; }
      
              public bool IsNotice => Body.StartsWith("[Notice]");
      
              public string CSS => Mine ? "sent" : "received";
          }
      }
      ```

   1. Agregue el marcado de la interfaz de usuario antes de la sección `@code` para interactuar con el cliente de SignalR.
      
      ```razor
      <h1>Blazor SignalR Chat Sample</h1>
      <hr />
      
      @if (!_isChatting)
      {
          <p>
              Enter your name to start chatting:
          </p>
      
          <input type="text" maxlength="32" @bind="@_username" />
          <button type="button" @onclick="@Chat"><span class="oi oi-chat" aria-hidden="true"></span> Chat!</button>
      
          // Error messages
          @if (_message != null)
          {
              <div class="invalid-feedback">@_message</div>
              <small id="emailHelp" class="form-text text-muted">@_message</small>
          }
      }
      else
      {
          // banner to show current user
          <div class="alert alert-secondary mt-4" role="alert">
              <span class="oi oi-person mr-2" aria-hidden="true"></span>
              <span>You are connected as <b>@_username</b></span>
              <button class="btn btn-sm btn-warning ml-md-auto" @onclick="@DisconnectAsync">Disconnect</button>
          </div>
          // display messages
          <div id="scrollbox">
              @foreach (var item in _messages)
              {
                  @if (item.IsNotice)
                  {
                      <div class="alert alert-info">@item.Body</div>
                  }
                  else
                  {
                      <div class="@item.CSS">
                          <div class="user">@item.Username</div>
                          <div class="msg">@item.Body</div>
                      </div>
                  }
              }
              <hr />
              <textarea class="input-lg" placeholder="enter your comment" @bind="@_newMessage"></textarea>
              <button class="btn btn-default" @onclick="@(() => SendAsync(_newMessage))">Send</button>
          </div>
      }
      ```

1. Actualice el componente `NavMenu.razor` para insertar un nuevo componente `NavLink` para vincularlo a la sala de chat en `NavMenuCssClass`.

   ```razor
   <li class="nav-item px-3">
       <NavLink class="nav-link" href="chatroom">
           <span class="oi oi-chat" aria-hidden="true"></span> Chat room
       </NavLink>
   </li>
   ```
   
1. Agregue algunas clases CSS al archivo `site.css` para dar estilo a los elementos de la interfaz de usuario de la página de chat.

   ```css
   /* improved for chat text box */
   textarea {
       border: 1px dashed #888;
       border-radius: 5px;
       width: 80%;
       overflow: auto;
       background: #f7f7f7
   }
   
   /* improved for speech bubbles */
   .received, .sent {
       position: relative;
       font-family: arial;
       font-size: 1.1em;
       border-radius: 10px;
       padding: 20px;
       margin-bottom: 20px;
   }
   
   .received:after, .sent:after {
       content: '';
       border: 20px solid transparent;
       position: absolute;
       margin-top: -30px;
   }
   
   .sent {
       background: #03a9f4;
       color: #fff;
       margin-left: 10%;
       top: 50%;
       text-align: right;
   }
   
   .received {
       background: #4CAF50;
       color: #fff;
       margin-left: 10px;
       margin-right: 10%;
   }
   
   .sent:after {
       border-left-color: #03a9f4;
       border-right: 0;
       right: -20px;
   }
   
   .received:after {
       border-right-color: #4CAF50;
       border-left: 0;
       left: -20px;
   }
   
   /* div within bubble for name */
   .user {
       font-size: 0.8em;
       font-weight: bold;
       color: #000;
   }
   
   .msg {
       /*display: inline;*/
   }
   ```

1. Presione <kbd>F5</kbd> para ejecutar la aplicación. Ahora puede iniciar el chat:

   [ ![Se muestra una animación de chat entre dos interlocutores. Ambos interlocutores se saludan.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat.gif) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat.gif#lightbox)
   
[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsblazor)

## <a name="publish-to-azure"></a>Publicar en Azure

Al implementar la aplicación Blazor en Azure App Service, se recomienda usar [Azure SignalR Service](/aspnet/core/signalr/scale#azure-signalr-service). Azure SignalR Service permite el escalado vertical de una aplicación Blazor Server a un gran número de conexiones de SignalR simultáneas. Además, los centros de datos de alto rendimiento y alcance global de SignalR Service son de gran ayuda a la hora de reducir la latencia ocasionada por la geografía.

> [!IMPORTANT]
> En una aplicación Blazor Server, los estados de la interfaz de usuario se mantienen en el lado del servidor, lo que significa que se requiere una sesión de servidor permanente para conservar el estado. Si hay un único servidor de aplicaciones, las sesiones permanentes se garantizan mediante diseño. Sin embargo, si hay varios servidores de aplicaciones, es posible que la negociación y conexión del cliente puedan ir a servidores diferentes, lo que puede dar lugar a una administración incoherente del estado de la interfaz de usuario en una aplicación Blazor. Por lo tanto, se recomienda habilitar las sesiones de servidor permanentes como se indica a continuación en *appsettings.json*:
>
> ```json
> "Azure:SignalR:ServerStickyMode": "Required"
> ```

1. Haga clic con el botón derecho en el proyecto y vaya a **Publicar**. Use la configuración siguiente:
   * **Destino**: Azure
   * **Destino específico**: se admiten todos los tipos de **Azure App Service**.
   * **App Service:** cree o seleccione la instancia de App Service.

   [ ![La animación muestra la selección de Azure como destino y, a continuación, la de Azure App Service como destino específico.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-profile.gif) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-profile.gif#lightbox)

1. Agregue la dependencia de Azure SignalR Service.

   Después de crear el perfil de publicación, puede ver un mensaje de recomendación para agregar Azure SignalR Service en **Dependencias del servicio**. Seleccione **Configurar** para crear una nueva o seleccione una instancia de Azure SignalR Service existente en el panel.

   [ ![Al publicar, se resalta el vínculo para configurar.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency.png#lightbox)

   La dependencia del servicio realizará las siguientes actividades para permitir que la aplicación cambie automáticamente a Azure SignalR Service cuando se ejecute en Azure.

   * Actualizar [`HostingStartupAssembly`](/aspnet/core/fundamentals/host/platform-specific-configuration) para usar Azure SignalR Service.
   * Agregar la referencia del paquete NuGet de Azure SignalR Service.
   * Actualizar las propiedades del perfil para guardar la configuración de dependencias.
   * Configurar el almacén de secretos según su elección.
   * Agregar la configuración de *appsettings.json* para que el destino de la aplicación sea Azure SignalR Service.

   [ ![En Resumen de cambios, se usan las casillas para seleccionar todas las dependencias.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency-summary.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency-summary.png#lightbox)

1. Publique la aplicación.

   La aplicación ya está lista para publicarla. Tras la finalización del proceso de publicación, la aplicación se inicia automáticamente en un explorador.
 
   > [!NOTE]
   > Puede que la aplicación necesite un tiempo para iniciarse debido a la latencia de inicio de la implementación de Azure App Service. Puede usar las herramientas del depurador del explorador (normalmente presionando <kbd>F12</kbd>) para asegurarse de que el tráfico se ha redirigido a Azure SignalR Service.

   [ ![El ejemplo de chat de SignalR para Blazor muestra un cuadro de texto para indicar el nombre y un botón de chat para iniciar un chat.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-azure.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-azure.png#lightbox)
   
[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsblazor)

## <a name="enable-azure-signalr-service-for-local-development"></a>Habilitación de Azure SignalR Service para el desarrollo local

1. Agregue una referencia al SDK de Azure SignalR mediante el comando siguiente.

   ```dotnetcli
   dotnet add package Microsoft.Azure.SignalR --version 1.5.1
   ```

1. Agregue una llamada a `AddAzureSingalR()` en `Startup.ConfigureServices()` como se indica a continuación.

   ```cs
   public void ConfigureServices(IServiceCollection services)
   {
       ...
       services.AddSignalR().AddAzureSignalR();
       ...
   }
   ```

1. Configure la cadena de conexión de Azure SignalR Service en *appsettings.json* o mediante la herramienta [Administrador de secretos](/aspnet/core/security/app-secrets?tabs=visual-studio#secret-manager).

> [!NOTE]
> El paso 2 se puede reemplazar por la configuración de [ensamblados de inicio de hospedaje](/aspnet/core/fundamentals/host/platform-specific-configuration) para que usen el SDK de SignalR.
>
> 1. Agregue la configuración para activar Azure SignalR Service en *appsettings.json*:
>
>     ```json
>     "Azure": {
>       "SignalR": {
>         "Enabled": true,
>         "ConnectionString": <your-connection-string>       
>       }
>     }
>    
>    ```
>
> 1. Configure el ensamblado de inicio de hospedaje para que utilice el SDK de Azure SignalR. Edite *launchSettings.json* y agregue una configuración como la del ejemplo siguiente dentro de `environmentVariables`:
>
>     ```json
>    "environmentVariables": {
>         ...,
>        "ASPNETCORE_HOSTINGSTARTUPASSEMBLIES": "Microsoft.Azure.SignalR"
>      }
>  
>     ```
>

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/asrs/qsblazor)

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar los recursos creados en este tutorial, elimine el grupo de recursos mediante Azure Portal.

## <a name="additional-resources"></a>Recursos adicionales

* [Introducción a ASP.NET Core Blazor](/aspnet/core/blazor)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear una sala de chat sencilla con la plantilla de la aplicación Blazor Server.
> * Trabajar con componentes de Razor.
> * Usar el control de eventos y el enlace de datos en los componentes de Razor.
> * Implementar rápidamente en Azure App Service con Visual Studio.
> * Migrar desde SignalR local a Azure SignalR Service.

Más información sobre la alta disponibilidad:
> [!div class="nextstepaction"]
> [Resistencia y recuperación ante desastres](signalr-concept-disaster-recovery.md)
