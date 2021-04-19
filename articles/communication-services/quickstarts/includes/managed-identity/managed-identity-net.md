---
ms.openlocfilehash: 424e70f4f3656be2196f4dbdfbddb852846b6897
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107307489"
---
## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `ManagedIdentitiesQuickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: `Program.cs`.

```console
dotnet new console -o ManagedIdentitiesQuickstartQuickstart
```

Cambie el directorio a la carpeta de la aplicación recién creada y use el comando `dotnet build` para compilar la aplicación.

```console
cd ManagedIdentitiesQuickstart
dotnet build
```

### <a name="install-the-sdk-packages"></a>Instalación de los paquetes del SDK

```console
dotnet add package Azure.Communication.Identity
dotnet add package Azure.Communication.Sms
dotnet add package Azure.Identity
```

### <a name="use-the-sdk-packages"></a>Uso de los paquetes del SDK

Agregue las siguientes directivas `using` a `Program.cs` para usar los SDK de Azure Storage y Azure Identity.

```csharp
using Azure.Identity;
using Azure.Communication.Identity;
using Azure.Communication.Sms;
using Azure.Core;
using Azure;
```

## <a name="create-a-defaultazurecredential"></a>Creación de una credencial DefaultAzureCredential

Usaremos [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) para este inicio rápido. Esta credencial es adecuada para entornos de producción y desarrollo. Como es necesaria para cada operación, vamos a crearla dentro de la clase `Program.cs`. Agregue lo siguiente en la parte superior del archivo.

```csharp
     private DefaultAzureCredential credential = new DefaultAzureCredential();
```

## <a name="issue-a-token-with-managed-identities"></a>Emisión de un token con identidades administradas

Ahora agregaremos código que usa la credencial creada para emitir un token de acceso VoIP. Llamaremos a este código más adelante.

```csharp
     public Response<AccessToken> CreateIdentityAndGetTokenAsync(Uri resourceEndpoint)
     {
          var client = new CommunicationIdentityClient(resourceEndpoint, this.credential);
          var identityResponse = client.CreateUser();
          var identity = identityResponse.Value;

          var tokenResponse = client.GetToken(identity, scopes: new[] { CommunicationTokenScope.VoIP });

          return tokenResponse;
     }
```

## <a name="send-an-sms-with-managed-identities"></a>Envío de un SMS con identidades administradas

Como otro ejemplo de uso de identidades administradas, agregaremos este código que usa la misma credencial para enviar un SMS:

```csharp
     public SmsSendResult SendSms(Uri resourceEndpoint, string from, string to, string message)
     {
          SmsClient smsClient = new SmsClient(resourceEndpoint, this.credential);
          SmsSendResult sendResult = smsClient.Send(
               from: from,
               to: to,
               message: message,
               new SmsSendOptions(enableDeliveryReport: true) // optional
          );

          return sendResult;
     }
```

## <a name="write-the-main-method"></a>Escritura del método principal

Su `Program.cs` ya debe tener un método principal, vamos a agregar código que llamará al código creado anteriormente para mostrar el uso de identidades administradas:

```csharp
     static void Main(string[] args)
     {
          // You can find your endpoint and access key from your resource in the Azure portal
          // e.g. "https://<RESOURCE_NAME>.communication.azure.com";
          Uri endpoint = new("https://<RESOURCENAME>.communication.azure.com/");

          // We need an instance of the program class to use within this method.
          Program instance = new();

          Console.WriteLine("Retrieving new Access Token, using Managed Identities");
          Response<AccessToken> response = instance.CreateIdentityAndGetTokenAsync(endpoint);
          Console.WriteLine($"Retrieved Access Token: {response.Value.Token}");

          Console.WriteLine("Sending SMS using Managed Identities");

          // You will need a phone number from your resource to send an SMS.
          SmsSendResult result = instance.SendSms(endpoint, "<Your ACS Phone Number>", "<The Phone Number you'd like to send the SMS to.>", "Hello from Managed Identities");
          Console.WriteLine($"Sms id: {result.MessageId}");
          Console.WriteLine($"Send Result Successful: {result.Successful}");
     }
```

El archivo `Program.cs` final debería tener este aspecto:

```csharp
class Program
     {
          private DefaultAzureCredential credential = new DefaultAzureCredential();
          static void Main(string[] args)
          {
               // You can find your endpoint and access key from your resource in the Azure portal
               // e.g. "https://<RESOURCE_NAME>.communication.azure.com";
               Uri endpoint = new("https://acstestingrifox.communication.azure.com/");

               // We need an instance of the program class to use within this method.
               Program instance = new();

               Console.WriteLine("Retrieving new Access Token, using Managed Identities");
               Response<AccessToken> response = instance.CreateIdentityAndGetTokenAsync(endpoint);
               Console.WriteLine($"Retrieved Access Token: {response.Value.Token}");

               Console.WriteLine("Sending SMS using Managed Identities");

               // You will need a phone number from your resource to send an SMS.
               SmsSendResult result = instance.SendSms(endpoint, "+18445504651", "+14256253982", "Hello from Managed Identities");
               Console.WriteLine($"Sms id: {result.MessageId}");
               Console.WriteLine($"Send Result Successful: {result.Successful}");
          }
          public Response<AccessToken> CreateIdentityAndGetTokenAsync(Uri resourceEndpoint)
          {
               var client = new CommunicationIdentityClient(resourceEndpoint, this.credential);
               var identityResponse = client.CreateUser();
               var identity = identityResponse.Value;

               var tokenResponse = client.GetToken(identity, scopes: new[] { CommunicationTokenScope.VoIP });

               return tokenResponse;
          }
          public SmsSendResult SendSms(Uri resourceEndpoint, string from, string to, string message)
          {
               SmsClient smsClient = new SmsClient(resourceEndpoint, this.credential);
               SmsSendResult sendResult = smsClient.Send(
                    from: from,
                    to: to,
                    message: message,
                    new SmsSendOptions(enableDeliveryReport: true) // optional
               );

               return sendResult;
          }
    }
```

## <a name="run-the-program"></a>Ejecución del programa

Ahora debería poder ejecutar la aplicación mediante `dotnet run` desde la carpeta de la aplicación. La salida debe ser similar a la siguiente:
```
Retrieving new Access Token, using Managed Identities
Retrieved Access Token: ey....
Sending SMS using Managed Identities
Sms id: Outgoing_..._noam
Send Result Successful: True
```
