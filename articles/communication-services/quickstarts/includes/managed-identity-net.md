---
ms.openlocfilehash: 8e80a08cf8846cecff0db69b3f6b081f360c43da
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106113492"
---
## <a name="add-managed-identity-to-your-communication-services-solution-net"></a>Incorporación de la identidad administrada a su solución de Communication Services (.NET)

### <a name="install-the-sdk-packages"></a>Instalación de los paquetes del SDK

```console
dotnet add package Azure.Communication.Identity  --version 1.0.0
dotnet add package Azure.Communication.Sms --version 1.0.0
dotnet add package Azure.Identity
```

### <a name="use-the-sdk-packages"></a>Uso de los paquetes del SDK

Agregue las siguientes directivas `using` al código para usar los SDK de Azure Storage y Azure Identity.

```csharp
using Azure.Identity;
using Azure.Communication.Identity;
using Azure.Communication.Sms;
using Azure.Core;
```

En los ejemplos siguientes se usa [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential). Esta credencial es adecuada para entornos de producción y desarrollo.

Las variables de entorno `AZURE_CLIENT_SECRET`, `AZURE_CLIENT_ID` y `AZURE_TENANT_ID` son necesarias para crear un objeto `DefaultAzureCredential`. Para crear una aplicación registrada en el entorno de desarrollo y configurar variables de entorno, consulte [Autorización del acceso con una identidad administrada](../managed-identity-from-cli.md).

### <a name="create-an-identity-and-issue-a-token-with-managed-identity"></a>Creación de una identidad y emisión de un token con una identidad administrada

En el ejemplo de código siguiente se muestra cómo crear un objeto de cliente de servicio con tokens de Azure Active Directory.

A continuación, use el cliente para emitir un token para un nuevo usuario:

```csharp
     public Response<AccessToken> CreateIdentityAndGetTokenAsync(Uri resourceEndpoint)
     {
          TokenCredential credential = new DefaultAzureCredential();

          // You can find your endpoint and access key from your resource in the Azure portal
          // "https://<RESOURCE_NAME>.communication.azure.com";

          var client = new CommunicationIdentityClient(resourceEndpoint, credential);
          var identityResponse = client.CreateUser();
          var identity = identityResponse.Value;

          var tokenResponse = client.GetToken(identity, scopes: new[] { CommunicationTokenScope.VoIP });

          return tokenResponse;
     }
```

### <a name="send-an-sms-with-managed-identity"></a>Envío de un SMS con identidad administrada

En el ejemplo de código siguiente se muestra cómo crear un objeto de cliente de servicio SMS con una identidad administrada y, a continuación, usar el cliente para enviar un mensaje SMS:

```csharp
     public SmsSendResult SendSms(Uri resourceEndpoint, string from, string to, string message)
     {
          TokenCredential credential = new DefaultAzureCredential();
          // You can find your endpoint and access key from your resource in the Azure portal
          // "https://<RESOURCE_NAME>.communication.azure.com";

          SmsClient smsClient = new SmsClient(resourceEndpoint, credential);
          SmsSendResult sendResult = smsClient.Send(
               from: from,
               to: to,
               message: message,
               new SmsSendOptions(enableDeliveryReport: true) // optional
          );

          return sendResult;
      }
```
