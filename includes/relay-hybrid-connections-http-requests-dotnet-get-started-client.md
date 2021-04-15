---
title: archivo de inclusión
description: archivo de inclusión
services: service-bus-relay
author: clemensv
ms.service: service-bus-relay
ms.topic: include
ms.date: 08/16/2018
ms.author: clemensv
ms.custom: include file
ms.openlocfilehash: ce29cd03de46e1d93d7f1f28f9f5184cd59a57e7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "79200109"
---
### <a name="create-a-console-application"></a>Creación de una aplicación de consola

Si ha desactivado la opción que indica que se requiere la autorización del cliente al crear la retransmisión, puede enviar solicitudes a la dirección URL de conexiones híbridas con un explorador. Para acceder a los puntos de conexión protegidos, debe crear y pasar un token en el encabezado `ServiceBusAuthorization`, que se muestra aquí.

En Visual Studio, cree un nuevo proyecto de **Aplicación de consola (.NET Framework)** .

### <a name="add-the-relay-nuget-package"></a>Adición del paquete Relay NuGet

1. Haga clic con el botón derecho en el proyecto recién creado y seleccione **Administrar paquetes NuGet**.
2. Seleccione la opción **Incluir versión preliminar**. 
3. Seleccione **Examinar** y, a continuación, busque **Microsoft.Azure.Relay**. En los resultados de la búsqueda, seleccione **Microsoft Azure Relay**.
4. Para la versión, seleccione **2.0.0-preview1-20180523**. 
5. Seleccione **Instalar** para completar la instalación. Cierre el cuadro de diálogo.

### <a name="write-code-to-send-requests"></a>Escritura de código para enviar solicitudes

1. En la parte superior del archivo Program.cs, reemplace las instrucciones `using` existentes por las siguientes instrucciones `using`:
   
    ```csharp
    using System;
    using System.IO;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Net.Http;
    using Microsoft.Azure.Relay;
    ```
2. Agregue constantes a la clase `Program` para los detalles de la conexión híbrida. Reemplace los marcadores de posición entre corchetes por los valores que obtuvo al crear la conexión híbrida. Asegúrese de utilizar el nombre de espacio de nombres completo.
   
    ```csharp
    private const string RelayNamespace = "{RelayNamespace}.servicebus.windows.net";
    private const string ConnectionName = "{HybridConnectionName}";
    private const string KeyName = "{SASKeyName}";
    private const string Key = "{SASKey}";
    ```
3. Agregue el siguiente método a la clase `Program`:
   
    ```csharp
    private static async Task RunAsync()
    {
        var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                KeyName, Key);
        var uri = new Uri(string.Format("https://{0}/{1}", RelayNamespace, ConnectionName));
        var token = (await tokenProvider.GetTokenAsync(uri.AbsoluteUri, TimeSpan.FromHours(1))).TokenString;
        var client = new HttpClient();
        var request = new HttpRequestMessage()
        {
            RequestUri = uri,
            Method = HttpMethod.Get,
        };
        request.Headers.Add("ServiceBusAuthorization", token);
        var response = await client.SendAsync(request);
        Console.WriteLine(await response.Content.ReadAsStringAsync());        Console.ReadLine();
    }
    ```
4. Agregue la siguiente línea de código al método `Main` de la clase `Program`.
   
    ```csharp
    RunAsync().GetAwaiter().GetResult();
    ```
   
    El archivo Program.cs debería tener el siguiente aspecto:
   
    ```csharp
    using System;
    using System.IO;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Net.Http;
    using Microsoft.Azure.Relay;
   
    namespace Client
    {
        class Program
        {
            private const string RelayNamespace = "{RelayNamespace}.servicebus.windows.net";
            private const string ConnectionName = "{HybridConnectionName}";
            private const string KeyName = "{SASKeyName}";
            private const string Key = "{SASKey}";
   
            static void Main(string[] args)
            {
                RunAsync().GetAwaiter().GetResult();
            }
   
            private static async Task RunAsync()
            {
               var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                KeyName, Key);
                var uri = new Uri(string.Format("https://{0}/{1}", RelayNamespace, ConnectionName));
                var token = (await tokenProvider.GetTokenAsync(uri.AbsoluteUri, TimeSpan.FromHours(1))).TokenString;
                var client = new HttpClient();
                var request = new HttpRequestMessage()
                {
                    RequestUri = uri,
                    Method = HttpMethod.Get,
                };
                request.Headers.Add("ServiceBusAuthorization", token);
                var response = await client.SendAsync(request);
                Console.WriteLine(await response.Content.ReadAsStringAsync());
            }
        }
    }
    ```

