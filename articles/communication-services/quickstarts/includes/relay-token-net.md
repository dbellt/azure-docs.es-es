---
title: archivo de inclusión
description: archivo de inclusión
services: Communication Services
author: shahen
manager: anvalent
ms.service: Communication Services
ms.subservice: Communication Services
ms.date: 05/21/2021
ms.topic: include
ms.custom: include file
ms.author: shahen
ms.openlocfilehash: fa3ecf3fa6b74bdbd52a9b077cde04c6f98dde86
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "111026948"
---
### <a name="prerequisite-check"></a>Comprobación de requisitos previos

- En una ventana de terminal o de comandos, ejecute el comando `dotnet` para comprobar que el SDK de .NET está instalado.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

1. En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `RelayTokenQuickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: **Program.cs**.

```console
dotnet new console -o RelayTokenQuickstart
```

1. Cambie el directorio a la carpeta de la aplicación recién creada y use el comando `dotnet build` para compilar la aplicación.

```console
cd RelayTokenQuickstart
dotnet build
```

### <a name="install-the-package"></a>Instalar el paquete

En el directorio de aplicaciones, instale el paquete de la biblioteca de identidades de Azure Communication Services y la biblioteca de Network Traversal para .NET usando el comando `dotnet add package`.

```console
dotnet add package Azure.Communication.Identity
dotnet add package Azure.Communication.NetworkTraversal
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

1. Abra el archivo **Program.cs** en un editor de texto.
2. Agregue una directiva `using` para incluir `Azure.Communication.Identity`, `System.Threading.Tasks` y `System.Net.Http`.
3. Actualice la declaración del método `Main` para admitir código asincrónico.

Este es el código:

```csharp
using System;
using Azure.Communication.Identity;
using Azure;
using System.Collections.Generic;
using Azure.Communication.NetworkTraversal;

namespace RelayTokenQuickstart
{
    class Program
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine("Azure Communication Services - User Relay Token Quickstart");

            // Quickstart code goes here
        }
    }
}
```

### <a name="authenticate-the-client"></a>Autenticar el cliente

Inicialice un objeto `CommunicationIdentityClient` con su cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso a partir de una variable de entorno. Si ha creado la variable de entorno después de haber iniciado la aplicación, deberá cerrar y volver a abrir el editor, el IDE o el shell que lo ejecuta para acceder a la variable.

Agregue el código siguiente al método `Main`:

```csharp
// This code demonstrates how to fetch your connection string
// from an environment variable.
string connectionString = Environment.GetEnvironmentVariable("COMMUNICATION_SERVICES_CONNECTION_STRING");
var client = new CommunicationIdentityClient(connectionString);
```

## <a name="create-an-identity"></a>Creación de una identidad

Azure Communication Services mantiene un directorio de identidades ligero. Use el método `createUser` para crear una nueva entrada en el directorio con un `Id` único. Almacene la identidad recibida con la asignación a los usuarios de la aplicación. Por ejemplo, almacenándolos en la base de datos del servidor de aplicaciones. La identidad es necesaria más adelante para emitir tokens de acceso.

```csharp
var identityResponse = await client.CreateUserAsync();
var identity = identityResponse.Value;
Console.WriteLine($"\nCreated an identity with ID: {identity.Id}");
```

### <a name="exchange-the-user-access-token-for-a-relay-token"></a>Intercambio del token de acceso de usuario por un token de retransmisión

Llame al servicio de token de Azure Communication para intercambiar el token de acceso de usuario por un token de servicio TURN.

```csharp
var relayClient = new CommunicationRelayClient("COMMUNICATION_SERVICES_CONNECTION_STRING");

Response<CommunicationRelayConfiguration> turnTokenResponse = await relayClient.GetRelayConfigurationAsync(identity);
DateTimeOffset turnTokenExpiresOn = turnTokenResponse.Value.ExpiresOn;
IReadOnlyList<CommunicationTurnServer> turnServers = turnTokenResponse.Value.TurnServers;
Console.WriteLine($"Expires On: {turnTokenExpiresOn}");
foreach (CommunicationTurnServer turnServer in turnServers)
{
    foreach (string url in turnServer.Urls)
    {
        Console.WriteLine($"TURN Url: {url}");
    }
    Console.WriteLine($"TURN Username: {turnServer.Username}");
    Console.WriteLine($"TURN Credential: {turnServer.Credential}");
}
```

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```console
dotnet run
```
