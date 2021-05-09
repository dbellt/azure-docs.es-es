---
ms.openlocfilehash: f5f9f30103381a8e91cd38bb7fa3547e6ca4c97e
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108157503"
---
## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- La versión más reciente de la [biblioteca cliente de .NET Core](https://dotnet.microsoft.com/download/dotnet-core) para su sistema operativo.
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../../create-communication-resource.md).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

- En una ventana de terminal o de comandos, ejecute `dotnet` para comprobar que la biblioteca cliente de .NET esté instalada.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `PhoneNumbersQuickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: **Program.cs**.

```console
dotnet new console -o PhoneNumbersQuickstart
```

Cambie el directorio a la carpeta de la aplicación recién creada y use el comando `dotnet build` para compilar la aplicación.

```console
cd PhoneNumbersQuickstart
dotnet build
```

### <a name="install-the-package"></a>Instalar el paquete

Mientras sigue en el directorio de aplicaciones, instale el paquete de la biblioteca cliente de PhoneNumbers de Azure Communication Services para .NET con el comando `dotnet add package`.

```console
dotnet add package Azure.Communication.PhoneNumbers --version 1.0.0
```

Agregue la directiva `using` a la parte superior de **Program.cs** para incluir el espacio de nombres.

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using Azure.Communication.PhoneNumbers;
```

Actualice la signatura de función `Main` para que sea asincrónica.

```csharp
static async Task Main(string[] args)
{
  ...
}
```

## <a name="authenticate-the-client"></a>Autenticar el cliente

Los clientes de PhoneNumbers se pueden autenticar mediante la cadena de conexión adquirida de los recursos de Azure Communication Services en [Azure Portal][azure_portal].

```csharp
// Get a connection string to our Azure Communication resource.
var connectionString = "<connection_string>";
var client = new PhoneNumbersClient(connectionString);
```

Los clientes de PhoneNumbers también tienen la opción de autenticarse con la autenticación de Azure Active Directory. Con esta opción, las variables de entorno `AZURE_CLIENT_SECRET` `AZURE_CLIENT_ID` y `AZURE_TENANT_ID` deben configurarse para la autenticación.

```csharp
// Get an endpoint to our Azure Communication resource.
var endpoint = new Uri("<endpoint_url>");
TokenCredential tokenCredential = new DefaultAzureCredential();
client = new PhoneNumbersClient(endpoint, tokenCredential);
```

## <a name="manage-phone-numbers"></a>Administración de números de teléfono

### <a name="search-for-available-phone-numbers"></a>Búsqueda de los números de teléfono disponibles

Para adquirir números de teléfono, primero debe buscar los que están disponibles. Para buscar números de teléfono, proporcione el código de área, el tipo de asignación, las [funcionalidades del número de teléfono](../../../concepts/telephony-sms/plan-solution.md#phone-number-capabilities-in-azure-communication-services), el [tipo de número de teléfono](../../../concepts/telephony-sms/plan-solution.md#phone-number-types-in-azure-communication-services) y la cantidad. Tenga en cuenta que proporcionar el código de área para el tipo de número de teléfono gratuito es opcional.

```csharp
var capabilities = new PhoneNumberCapabilities(calling:PhoneNumberCapabilityType.None, sms:PhoneNumberCapabilityType.Outbound);
var searchOptions = new PhoneNumberSearchOptions { AreaCode = "833", Quantity = 1 };

var searchOperation = await client.StartSearchAvailablePhoneNumbersAsync("US", PhoneNumberType.TollFree, PhoneNumberAssignmentType.Application, capabilities, searchOptions);
await searchOperation.WaitForCompletionAsync();
```

### <a name="purchase-phone-numbers"></a>Compra de números de teléfono

El resultado de la búsqueda de números de teléfono es un valor de `PhoneNumberSearchResult`. Este valor contiene el valor de `SearchId`, que se puede pasar a la API de compra de números de para obtener los números en la búsqueda. Tenga en cuenta que, al llamar a la API de compra de números de teléfono, se realizará un cargo a su cuenta de Azure.

```csharp
var purchaseOperation = await client.StartPurchasePhoneNumbersAsync(searchOperation.Value.SearchId);
await purchaseOperation.WaitForCompletionResponseAsync();
```

### <a name="get-phone-numbers"></a>Obtención de números de teléfono

Después de comprar un número, puede recuperarlo del cliente.
```csharp
var getPhoneNumberResponse = await client.GetPurchasedPhoneNumberAsync("+14255550123");
Console.WriteLine($"Phone number: {getPhoneNumberResponse.Value.PhoneNumber}, country code: {getPhoneNumberResponse.Value.CountryCode}");
```

También puede recuperar todos los números de teléfono comprados.
``` csharp
var purchasedPhoneNumbers = client.GetPurchasedPhoneNumbersAsync();
await foreach (var purchasedPhoneNumber in purchasedPhoneNumbers)
{
    Console.WriteLine($"Phone number: {purchasedPhoneNumber.PhoneNumber}, country code: {purchasedPhoneNumber.CountryCode}");
}
```

### <a name="update-phone-number-capabilities"></a>Actualización de las funcionalidades del número de teléfono

Con un número comprado, puede actualizar las funcionalidades.

````csharp
var updateCapabilitiesOperation = await client.StartUpdateCapabilitiesAsync("+14255550123", calling: PhoneNumberCapabilityType.Outbound, sms: PhoneNumberCapabilityType.InboundOutbound);
await updateCapabilitiesOperation.WaitForCompletionAsync();
````


### <a name="release-phone-number"></a>Descarte de un número de teléfono

Puede liberar los números de teléfono comprados.

````csharp
var releaseOperation = await client.StartReleasePhoneNumberAsync("+14255550123");
await releaseOperation.WaitForCompletionResponseAsync();
````

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```console
dotnet run
```

## <a name="sample-code"></a>Código de ejemplo

Puede descargar la aplicación de ejemplo de [GitHub](https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/PhoneNumbers).
