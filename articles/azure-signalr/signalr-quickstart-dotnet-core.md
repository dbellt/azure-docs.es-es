---
title: Guía de inicio rápido para aprender a usar Azure SignalR Service
description: Una guía de inicio rápido para usar Azure SignalR Service para crear un salón de chat con aplicaciones de ASP.NET Core MVC.
author: sffamily
ms.service: signalr
ms.devlang: dotnet
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.date: 09/28/2020
ms.author: zhshang
ms.openlocfilehash: 9099a8620f6e4d87ec38c10226d94b1b3cd3462f
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107865988"
---
# <a name="quickstart-create-a-chat-room-by-using-signalr-service"></a>Inicio rápido: Creación de un salón de chat con SignalR Service

Azure SignalR Service es un servicio de Azure que ayuda a los desarrolladores a compilar fácilmente aplicaciones web con características en tiempo real. Este servicio se basó originalmente en [SignalR para ASP.NET Core 2.1](/aspnet/core/signalr/introduction?preserve-view=true&view=aspnetcore-2.1), pero ahora es compatible con versiones posteriores.

En este artículo se muestra cómo empezar a trabajar con Azure SignalR Service. En este inicio rápido, creará una aplicación de chat con una aplicación web de ASP.NET Core MVC. Esta aplicación establecerá una conexión con el recurso de Azure SignalR Service para habilitar las actualizaciones de contenido en tiempo real. Hospedará la aplicación web localmente y se conectará con varios clientes de explorador. Cada cliente podrá insertar actualizaciones de contenido en todos los demás clientes. 

Puede usar cualquier editor de código para realizar los pasos de esta guía de inicio rápido. Una opción es [Visual Studio Code](https://code.visualstudio.com/), que está disponible en las plataformas Windows, macOS y Linux.

El código de este tutorial está disponible para su descarga en el [repositorio de GitHub AzureSignalR-samples](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/ChatRoom). Además, puede crear los recursos de Azure usados en este inicio rápido siguiendo el artículo [Creación de una instancia de SignalR Service](scripts/signalr-cli-create-service.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note-dotnet.md)]

## <a name="prerequisites"></a>Prerrequisitos

* Instale el [SDK de .NET Core](https://dotnet.microsoft.com/download).
* Descargue o clone el repositorio de GitHub [AzureSignalR-sample](https://github.com/aspnet/AzureSignalR-samples). 

¿Tiene problemas? Consulte la [guía de solución de problemas](signalr-howto-troubleshoot-guide.md) o [póngase en contacto con nosotros](https://aka.ms/asrs/qsnetcore).

## <a name="create-an-azure-signalr-resource"></a>Creación de un recurso de Azure SignalR

[!INCLUDE [azure-signalr-create](../../includes/signalr-create.md)]

¿Tiene problemas? Consulte la [guía de solución de problemas](signalr-howto-troubleshoot-guide.md) o [póngase en contacto con nosotros](https://aka.ms/asrs/qsnetcore).

## <a name="create-an-aspnet-core-web-app"></a>Cree una aplicación web ASP.NET Core

En esta sección, usará la [interfaz de la línea de comandos (CLI) de .NET Core](/dotnet/core/tools/) para crear un proyecto de aplicación web de ASP.NET Core MVC. La ventaja de usar la CLI de .NET Core frente a Visual Studio es que está disponible en las plataformas Windows, macOS y Linux. 

1. Cree una carpeta para el proyecto. En este inicio rápido se usa la carpeta *E:\Testing\chattest*.

2. En la nueva carpeta, ejecute el siguiente comando para crear el proyecto:

    ```dotnetcli
    dotnet new mvc
    ```

¿Tiene problemas? Consulte la [guía de solución de problemas](signalr-howto-troubleshoot-guide.md) o [póngase en contacto con nosotros](https://aka.ms/asrs/qsnetcore).

## <a name="add-secret-manager-to-the-project"></a>Adición de Secret Manager al proyecto

En esta sección, agregará la [herramienta Secret Manager](/aspnet/core/security/app-secrets) al proyecto. La herramienta Secret Manager almacena información confidencial para el trabajo de desarrollo fuera de su árbol de proyecto. Este enfoque ayuda a evitar el uso compartido por accidente de secretos de la aplicación en el código fuente.

1. Abra el archivo *.csproj*. Agregue un elemento `DotNetCliToolReference` para incluir *Microsoft.Extensions.SecretManager.Tools*. Agregue también un elemento `UserSecretsId`, como se muestra en el código siguiente para *chattest.csproj* y guarde el archivo.

    ```xml
    <Project Sdk="Microsoft.NET.Sdk.Web">

    <PropertyGroup>
        <TargetFramework>netcoreapp3.1</TargetFramework>
        <UserSecretsId>SignalRChatRoomEx</UserSecretsId>
    </PropertyGroup>

    <ItemGroup>
        <DotNetCliToolReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.4" />
        <DotNetCliToolReference Include="Microsoft.Extensions.SecretManager.Tools" Version="2.0.2" />
    </ItemGroup>

    </Project>
    ```

¿Tiene problemas? Consulte la [guía de solución de problemas](signalr-howto-troubleshoot-guide.md) o [póngase en contacto con nosotros](https://aka.ms/asrs/qsnetcore).

## <a name="add-azure-signalr-to-the-web-app"></a>Adición de Azure SignalR a la aplicación web

1. Para agregar una referencia al paquete NuGet `Microsoft.Azure.SignalR`, ejecute el comando siguiente:

    ```dotnetcli
    dotnet add package Microsoft.Azure.SignalR
    ```

2. Ejecute el siguiente comando para restaurar los paquetes para el proyecto:

    ```dotnetcli
    dotnet restore
    ```

3. Agregue un secreto llamado *Azure:SignalR:ConnectionString* a Secret Manager. 

    Este secreto contendrá la cadena de conexión para tener acceso al recurso de SignalR Service. *Azure__SignalR__ConnectionString* es la clave de configuración predeterminada que busca SignalR con el fin de establecer una conexión. Sustituya el valor del siguiente comando por la cadena de conexión del recurso de SignalR Service.

    Este comando debe ejecutarse en el mismo directorio que el archivo *.csproj*.

    ```dotnetcli
    dotnet user-secrets set Azure:SignalR:ConnectionString "<Your connection string>"
    ```

    Secret Manager solo se usará para probar la aplicación web mientras se hospeda localmente. En un tutorial posterior, se implementará la aplicación web de chat en Azure. Una vez que la aplicación web se haya implementado en Azure, usará una configuración de la aplicación en lugar de almacenar la cadena de conexión con Secret Manager.

    A este secreto se accede con la API de configuración. Un signo de dos puntos (:) funciona en el nombre de configuración con la API de configuración en todas las plataformas compatibles. Consulte [Configuración en ASP.NET Core](/dotnet/core/extensions/configuration-providers#environment-variable-configuration-provider).


4. Abra *Startup.cs* y actualice el método `ConfigureServices` para usar Azure SignalR Service mediante una llamada a los métodos `AddSignalR()` y `AddAzureSignalR()`:

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSignalR()
                .AddAzureSignalR();
    }
    ```

    Al no pasar un parámetro a `AddAzureSignalR()`, este código usa la clave de configuración predeterminada para la cadena de conexión del recurso de SignalR Service. La clave de la configuración predeterminada es *Azure:SignalR:ConnectionString*.

5. En *Startup.cs*, actualice el método `Configure` reemplazándolo por el código siguiente.

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        app.UseRouting();
        app.UseFileServer();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapHub<ChatHub>("/chat");
        });
    }
    ```

### <a name="add-a-hub-class"></a>Adición de una clase de hub

En SignalR, un concentrador es un componente básico que expone un conjunto de métodos al que se puede llamar desde el cliente. En esta sección, define una clase de hub con dos métodos:

* `Broadcast`: este método difunde un mensaje a todos los clientes.
* `Echo`: este método devuelve un mensaje al autor de la llamada.

Los dos métodos usan la interfaz `Clients` proporcionada por el SDK de ASP.NET Core SignalR. Esta interfaz le da acceso a todos los clientes conectados, así puede insertar contenido en los clientes.

1. En el directorio del proyecto, agregue una nueva carpeta denominada *Hub*. Agregue un nuevo archivo de código de centro denominado *ChatHub.cs* a la nueva carpeta.

2. Agregue el código siguiente a *ChatHub.cs* para definir la clase de centro y guarde el archivo.

    Actualice el espacio de nombres para esta clase si ha usado un nombre de proyecto que difiere de *SignalR.Mvc*.

    ```csharp
    using Microsoft.AspNetCore.SignalR;
    using System.Threading.Tasks;
    
    namespace SignalR.Mvc
    {
        public class ChatHub : Hub
        {
            public Task BroadcastMessage(string name, string message) =>
                Clients.All.SendAsync("broadcastMessage", name, message);
    
            public Task Echo(string name, string message) =>
                Clients.Client(Context.ConnectionId)
                       .SendAsync("echo", name, $"{message} (echo from server)");
        }
    }
    ```

### <a name="add-the-client-interface-for-the-web-app"></a>Adición de la interfaz de cliente para la aplicación web

La interfaz de usuario del cliente en esta aplicación de salón de chat se compondrá de HTML y JavaScript en un archivo llamado *index.html* en el directorio *wwwroot*.

Copie el archivo *css/site.css* de la carpeta *wwwroot* del [repositorio de ejemplos](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/ChatRoom/wwwroot). Reemplace el archivo *css/site.css* por el que ha copiado.

Este es el código principal de *index.html*:

Cree un archivo en el directorio *wwwroot* denominado *index.html* y copie y pegue el siguiente código HTML en el archivo recién creado:

```html
<!DOCTYPE html>
<html>
<head>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" rel="stylesheet" />
    <link href="css/site.css" rel="stylesheet" />
    <title>Azure SignalR Group Chat</title>
</head>
<body>
    <h2 class="text-center" style="margin-top: 0; padding-top: 30px; padding-bottom: 30px;">Azure SignalR Group Chat</h2>
    <div class="container" style="height: calc(100% - 110px);">
        <div id="messages" style="background-color: whitesmoke; "></div>
        <div style="width: 100%; border-left-style: ridge; border-right-style: ridge;">
            <textarea id="message"
                      style="width: 100%; padding: 5px 10px; border-style: hidden;"
                      placeholder="Type message and press Enter to send..."></textarea>
        </div>
        <div style="overflow: auto; border-style: ridge; border-top-style: hidden;">
            <button class="btn-warning pull-right" id="echo">Echo</button>
            <button class="btn-success pull-right" id="sendmessage">Send</button>
        </div>
    </div>
    <div class="modal alert alert-danger fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <div>Connection Error...</div>
                    <div><strong style="font-size: 1.5em;">Hit Refresh/F5</strong> to rejoin. ;)</div>
                </div>
            </div>
        </div>
    </div>

    <!--Reference the SignalR library. -->
    <script src="https://cdn.jsdelivr.net/npm/@microsoft/signalr@3.1.8/dist/browser/signalr.min.js"></script>

    <!--Add script to update the page and send messages.-->
    <script type="text/javascript">
        document.addEventListener('DOMContentLoaded', function () {

            const generateRandomName = () =>
                Math.random().toString(36).substring(2, 10);

            let username = generateRandomName();
            const promptMessage = 'Enter your name:';
            do {
                username = prompt(promptMessage, username);
                if (!username || username.startsWith('_') || username.indexOf('<') > -1 || username.indexOf('>') > -1) {
                    username = '';
                    promptMessage = 'Invalid input. Enter your name:';
                }
            } while (!username)

            const messageInput = document.getElementById('message');
            messageInput.focus();

            function createMessageEntry(encodedName, encodedMsg) {
                var entry = document.createElement('div');
                entry.classList.add("message-entry");
                if (encodedName === "_SYSTEM_") {
                    entry.innerHTML = encodedMsg;
                    entry.classList.add("text-center");
                    entry.classList.add("system-message");
                } else if (encodedName === "_BROADCAST_") {
                    entry.classList.add("text-center");
                    entry.innerHTML = `<div class="text-center broadcast-message">${encodedMsg}</div>`;
                } else if (encodedName === username) {
                    entry.innerHTML = `<div class="message-avatar pull-right">${encodedName}</div>` +
                        `<div class="message-content pull-right">${encodedMsg}<div>`;
                } else {
                    entry.innerHTML = `<div class="message-avatar pull-left">${encodedName}</div>` +
                        `<div class="message-content pull-left">${encodedMsg}<div>`;
                }
                return entry;
            }

            function bindConnectionMessage(connection) {
                var messageCallback = function (name, message) {
                    if (!message) return;
                    var encodedName = name;
                    var encodedMsg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
                    var messageEntry = createMessageEntry(encodedName, encodedMsg);

                    var messageBox = document.getElementById('messages');
                    messageBox.appendChild(messageEntry);
                    messageBox.scrollTop = messageBox.scrollHeight;
                };
                connection.on('broadcastMessage', messageCallback);
                connection.on('echo', messageCallback);
                connection.onclose(onConnectionError);
            }

            function onConnected(connection) {
                console.log('connection started');
                connection.send('broadcastMessage', '_SYSTEM_', username + ' JOINED');
                document.getElementById('sendmessage').addEventListener('click', function (event) {
                    if (messageInput.value) {
                        connection.send('broadcastMessage', username, messageInput.value);
                    }

                    messageInput.value = '';
                    messageInput.focus();
                    event.preventDefault();
                });
                document.getElementById('message').addEventListener('keypress', function (event) {
                    if (event.keyCode === 13) {
                        event.preventDefault();
                        document.getElementById('sendmessage').click();
                        return false;
                    }
                });
                document.getElementById('echo').addEventListener('click', function (event) {
                    connection.send('echo', username, messageInput.value);

                    messageInput.value = '';
                    messageInput.focus();
                    event.preventDefault();
                });
            }

            function onConnectionError(error) {
                if (error && error.message) {
                    console.error(error.message);
                }
                var modal = document.getElementById('myModal');
                modal.classList.add('in');
                modal.style = 'display: block;';
            }

            const connection = new signalR.HubConnectionBuilder()
                .withUrl('/chat')
                .build();
            bindConnectionMessage(connection);
            connection.start()
                .then(() => onConnected(connection))
                .catch(error => console.error(error.message));
        });
    </script>
</body>
</html>
```

El código de *index.html* llama a `HubConnectionBuilder.build()` para realizar una conexión HTTP al recurso de Azure SignalR.

Si la conexión es correcta, esa conexión se pasa a `bindConnectionMessage`, que agrega controladores de eventos para las inserciones de contenido entrante al cliente. 

`HubConnection.start()` inicia la comunicación con el hub. A continuación, `onConnected()` agrega los controladores de eventos de botón. Estos controladores utilizan la conexión para permitir que este cliente inserte las actualizaciones de contenido en todos los clientes conectados.

## <a name="add-a-development-runtime-profile"></a>Adición de un perfil de desarrollo en tiempo de ejecución

En esta sección, agregará un entorno de desarrollo en tiempo de ejecución para ASP.NET Core. Para más información, consulte [Usar varios entornos de ASP.NET Core](/aspnet/core/fundamentals/environments).

1. Cree una carpeta llamada *Propiedades* en el proyecto.

2. Agregue un nuevo archivo llamado *launchSettings.json* a la carpeta, con el siguiente contenido, y guarde el archivo.

    ```json
    {
        "profiles" : {
            "ChatRoom": {
                "commandName": "Project",
                "launchBrowser": true,
                "environmentVariables": {
                    "ASPNETCORE_ENVIRONMENT": "Development"
                },
                "applicationUrl": "http://localhost:5000/"
            }
        }
    }
    ```

¿Tiene problemas? Consulte la [guía de solución de problemas](signalr-howto-troubleshoot-guide.md) o [póngase en contacto con nosotros](https://aka.ms/asrs/qsnetcore).

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Para compilar la aplicación mediante la CLI de .NET Core, ejecute el siguiente comando en el shell de comandos:

    ```dotnetcli
    dotnet build
    ```

1. Cuando la compilación haya finalizado correctamente, ejecute el siguiente comando para ejecutar la aplicación web localmente:

    ```dotnetcli
    dotnet run
    ```

    La aplicación se hospedará localmente en el puerto 5000, como está configurado en el perfil de desarrollo en tiempo de ejecución:

    ```output
    info: Microsoft.Hosting.Lifetime[0]
          Now listening on: https://localhost:5001
    info: Microsoft.Hosting.Lifetime[0]
          Now listening on: http://localhost:5000
    info: Microsoft.Hosting.Lifetime[0]
          Application started. Press Ctrl+C to shut down.
    info: Microsoft.Hosting.Lifetime[0]
          Hosting environment: Development
    info: Microsoft.Hosting.Lifetime[0]
          Content root path: E:\Testing\chattest
    ```

1. Abra dos ventanas de explorador. En cada explorador, vaya a `http://localhost:5000`. Se le pide que escriba su nombre. Escriba un nombre de cliente para ambos clientes y pruebe a insertar el contenido del mensaje entre ambos clientes con el botón **Enviar**.

    ![Ejemplo de un chat de grupo de Azure SignalR](media/signalr-quickstart-dotnet-core/signalr-quickstart-complete-local.png)

¿Tiene problemas? Consulte la [guía de solución de problemas](signalr-howto-troubleshoot-guide.md) o [póngase en contacto con nosotros](https://aka.ms/asrs/qsnetcore).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a seguir con el tutorial siguiente, puede mantener los recursos creados en este inicio rápido y volverlos a usar.

Si ya ha terminado con la aplicación de ejemplo del inicio rápido, puede eliminar los recursos de Azure que se han creado para evitar cargos. 

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible e incluye todos los recursos de ese grupo. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados. Si ha creado los recursos para hospedar este ejemplo en un grupo de recursos existente que contiene recursos que quiere conservar, puede eliminar cada recurso individualmente de sus hojas, en lugar de eliminar el grupo de recursos.

Inicie sesión en [Azure Portal](https://portal.azure.com) y después seleccione **Grupos de recursos**.

Escriba el nombre del grupo de recursos en el cuadro de texto **Filtrar por nombre**. Las instrucciones de esta guía de inicio rápido usan un grupo de recursos llamado *SignalRTestResources*. En el grupo de recursos, en la lista de resultados, seleccione los puntos suspensivos ( **...** ) > **Eliminar grupo de recursos**.

![Selecciones para eliminar un grupo de recursos](./media/signalr-quickstart-dotnet-core/signalr-delete-resource-group.png)

Se le pedirá que confirme la eliminación del grupo de recursos. Escriba el nombre del grupo de recursos para confirmar y seleccione **Eliminar**.

Transcurridos unos instantes, el grupo de recursos y todos sus recursos se eliminan.

¿Tiene problemas? Consulte la [guía de solución de problemas](signalr-howto-troubleshoot-guide.md) o [póngase en contacto con nosotros](https://aka.ms/asrs/qsnetcore).

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un nuevo recurso de Azure SignalR Service. Luego, lo ha usado con una aplicación web ASP.NET Core para insertar actualizaciones de contenido en tiempo real en varios clientes conectados. Para más información sobre cómo usar Azure SignalR Service, continúe con el siguiente tutorial que muestra cómo realizar la autenticación.

> [!div class="nextstepaction"]
> [Autenticación de Azure SignalR Service](./signalr-concept-authenticate-oauth.md)
