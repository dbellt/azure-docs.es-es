---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: tomaschladek
manager: nmurav
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: tchladek
ms.openlocfilehash: 1d846a282eba516a69460e020cd22c80f845e2f5
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111430929"
---
> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/AccessTokensQuickstart)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- La versión más reciente del [SDK de .NET Core](https://dotnet.microsoft.com/download/dotnet-core) para su sistema operativo.
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../create-communication-resource.md).

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `AccessTokensQuickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: **Program.cs**.

```console
dotnet new console -o AccessTokensQuickstart
```

Cambie el directorio a la carpeta de la aplicación recién creada y use el comando `dotnet build` para compilar la aplicación.

```console
cd AccessTokensQuickstart
dotnet build
```

### <a name="install-the-package"></a>Instalar el paquete

En el directorio de aplicaciones, instale el paquete de la biblioteca de identidades de Azure Communication Services para .NET usando el comando `dotnet add package`.

```console
dotnet add package Azure.Communication.Identity --version 1.0.0
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

1. Abra el archivo **Program.cs** en un editor de texto.
1. Agregue una directiva `using` para incluir el espacio de nombres `Azure.Communication.Identity`.
1. Actualice la declaración del método `Main` para admitir código asincrónico.

Use el código siguiente para empezar:

```csharp
using System;
using Azure;
using Azure.Core;
using Azure.Communication.Identity;

namespace AccessTokensQuickstart
{
    class Program
    {
        static async System.Threading.Tasks.Task Main(string[] args)
        {
            Console.WriteLine("Azure Communication Services - Access Tokens Quickstart");

            // Quickstart code goes here
        }
    }
}
```
## <a name="authenticate-the-client"></a>Autenticar el cliente

Inicialice un objeto `CommunicationIdentityClient` con su cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Aprenda a [administrar la cadena de conexión del recurso](../create-communication-resource.md#store-your-connection-string).

Agregue el código siguiente al método `Main`:

```csharp
// This code demonstrates how to fetch your connection string
// from an environment variable.
string connectionString = Environment.GetEnvironmentVariable("COMMUNICATION_SERVICES_CONNECTION_STRING");
var client = new CommunicationIdentityClient(connectionString);
```

Como alternativa, puede separar el punto de conexión y la clave de acceso.
```csharp
// This code demonstrates how to fetch your endpoint and access key
// from an environment variable.
string endpoint = Environment.GetEnvironmentVariable("COMMUNICATION_SERVICES_ENDPOINT");
string accessKey = Environment.GetEnvironmentVariable("COMMUNICATION_SERVICES_ACCESSKEY");
var client = new CommunicationIdentityClient(new Uri(endpoint), new AzureKeyCredential(accessKey));
```

Si ha configurado una identidad administrada, consulte [Uso de identidades administradas](../managed-identity.md); también puede autenticarse con la identidad administrada.
```csharp
TokenCredential tokenCredential = new DefaultAzureCredential();
var client = new CommunicationIdentityClient(new Uri(endpoint), tokenCredential);
```

## <a name="create-an-identity"></a>Creación de una identidad

Azure Communication Services mantiene un directorio de identidades ligero. Use el método `createUser` para crear una nueva entrada en el directorio con un `Id` único. Almacene la identidad recibida con la asignación a los usuarios de la aplicación. Por ejemplo, almacenándolos en la base de datos del servidor de aplicaciones. La identidad es necesaria más adelante para emitir tokens de acceso.

```csharp
var identityResponse = await client.CreateUserAsync();
var identity = identityResponse.Value;
Console.WriteLine($"\nCreated an identity with ID: {identity.Id}");
```

## <a name="issue-identity-access-tokens"></a>Emisión de tokens de acceso de identidad

Use el método `GetToken` para emitir un token de acceso para la identidad de Communication Services existente. El parámetro `scopes` define un conjunto de primitivas que autorizará este token de acceso. Consulte la [lista de plataformas admitidas](../../concepts/authentication.md). Se puede crear una nueva instancia del parámetro `communicationUser` en función de la representación de cadena de la identidad de Azure Communication Services.

```csharp
// Issue an access token with the "voip" scope for an identity
var tokenResponse = await client.GetTokenAsync(identity, scopes: new [] { CommunicationTokenScope.VoIP });
var token =  tokenResponse.Value.Token;
var expiresOn = tokenResponse.Value.ExpiresOn;
Console.WriteLine($"\nIssued an access token with 'voip' scope that expires at {expiresOn}:");
Console.WriteLine(token);
```

Los tokens de acceso son credenciales de corta duración que deben volver a emitirse. No hacerlo puede provocar una interrupción de la experiencia de los usuarios respecto a la aplicación. La propiedad de respuesta `expiresOn` indica la duración del token de acceso.

## <a name="create-an-identity-and-issue-an-access-token-within-the-same-request"></a>Cree una identidad y emita un token de acceso dentro de la misma solicitud.

Use el método `CreateUserAndTokenAsync` para crear una identidad de Communication Services y emitir un token de acceso para ella. El parámetro `scopes` define un conjunto de primitivas que autorizará este token de acceso. Consulte la [lista de plataformas admitidas](../../concepts/authentication.md).

```csharp
// Issue an identity and an access token with the "voip" scope for the new identity
var identityAndTokenResponse = await client.CreateUserAndTokenAsync(scopes: new[] { CommunicationTokenScope.VoIP });
var identity = identityAndTokenResponse.Value.User;
var token = identityAndTokenResponse.Value.AccessToken.Token;
var expiresOn = identityAndTokenResponse.Value.AccessToken.ExpiresOn;

Console.WriteLine($"\nCreated an identity with ID: {identity.Id}");
Console.WriteLine($"\nIssued an access token with 'voip' scope that expires at {expiresOn}:");
Console.WriteLine(token);
```

## <a name="refresh-access-tokens"></a>Tokens de acceso de actualización

Para actualizar un token de acceso, pase una instancia del objeto `CommunicationUserIdentifier` en `GetTokenAsync`. Si ha almacenado este `Id` y necesita crear un objeto `CommunicationUserIdentifier`, puede hacerlo pasando el `Id` almacenado al constructor `CommunicationUserIdentifier` como se indica a continuación:

```csharp
var identityToRefresh = new CommunicationUserIdentifier(identity.Id);
var tokenResponse = await client.GetTokenAsync(identityToRefresh, scopes: new [] { CommunicationTokenScope.VoIP });
```

## <a name="revoke-access-tokens"></a>Revocación de los tokens de acceso

En algunos casos, puede revocar explícitamente los tokens de acceso. Por ejemplo, cuando el usuario de una aplicación cambia la contraseña que usa para autenticarse en el servicio. El método `RevokeTokensAsync` invalida todos los tokens de acceso activos emitidos para la identidad.

```csharp
await client.RevokeTokensAsync(identity);
Console.WriteLine($"\nSuccessfully revoked all access tokens for identity with ID: {identity.Id}");
```

## <a name="delete-an-identity"></a>Eliminación de una identidad

La eliminación de una identidad revoca todos los tokens de acceso activos e impide que se emitan tokens de acceso posteriores para las identidades. También quita todo el contenido conservado asociado a la identidad.

```csharp
await client.DeleteUserAsync(identity);
Console.WriteLine($"\nDeleted the identity with ID: {identity.Id}");
```

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```console
dotnet run
```
