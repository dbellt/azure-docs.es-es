---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: peiliu
manager: rejooyan
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/11/2021
ms.topic: include
ms.custom: include file
ms.author: peiliu
ms.openlocfilehash: 177c433fcc1fb990a610405c8343340314816966
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111430369"
---
Para empezar a usar Azure Communication Services, utilice el SDK de SMS de Communication Services para C# para enviar mensajes SMS.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/SendSMS)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- La versión más reciente del [SDK de .NET Core](https://dotnet.microsoft.com/download/dotnet-core) para su sistema operativo.
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- Un número de teléfono habilitado para SMS. [Obtención de un número de teléfono](../get-phone-number.md).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

- En una ventana de terminal o de comandos, ejecute el comando `dotnet` para comprobar que el SDK de .NET está instalado.
- Para ver los números de teléfono asociados a su recurso de Communication Services, inicie sesión en [Azure Portal](https://portal.azure.com/), busque el recurso de Communication Services y abra la pestaña **números de teléfono** en el panel de navegación izquierdo.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `SmsQuickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: **Program.cs**.

```console
dotnet new console -o SmsQuickstart
```

Cambie el directorio a la carpeta de la aplicación recién creada y use el comando `dotnet build` para compilar la aplicación.

```console
cd SmsQuickstart
dotnet build
```

### <a name="install-the-package"></a>Instalar el paquete

Mientras sigue en el directorio de la aplicación, instale el paquete del SDK de SMS de Azure Communication Services para .NET con el comando `dotnet add package`.

```console
dotnet add package Azure.Communication.Sms --version 1.0.0
```

Agregue la directiva `using` a la parte superior de **Program.cs** para incluir el espacio de nombres `Azure.Communication`.

```csharp

using System;
using System.Collections.Generic;

using Azure;
using Azure.Communication;
using Azure.Communication.Sms;

```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales del SDK de SMS de Azure Communication Services para C#.

| Nombre                                       | Descripción                                                                                                                                                       |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SmsClient     | Esta clase es necesaria para la funcionalidad de los SMS. Cree una instancia de esta clase con la información de suscripción y úsela para enviar mensajes de texto.                           |
| SmsSendOptions | Esta clase proporciona opciones para configurar los informes de entrega. Si enable_delivery_report se establece en True, se emitirá un evento cuando la entrega se realice correctamente |
| SmsSendResult               | Esta clase contiene el resultado del servicio SMS.                                          |

## <a name="authenticate-the-client"></a>Autenticar el cliente

 Abra **Program.cs** en un editor de texto y reemplace el cuerpo del método `Main` por el código para inicializar `SmsClient` con la cadena de conexión. El código siguiente recupera la cadena de conexión para el recurso de una variable de entorno denominada `COMMUNICATION_SERVICES_CONNECTION_STRING`. Aprenda a [administrar la cadena de conexión del recurso](../../create-communication-resource.md#store-your-connection-string).


```csharp
// This code demonstrates how to fetch your connection string
// from an environment variable.
string connectionString = Environment.GetEnvironmentVariable("COMMUNICATION_SERVICES_CONNECTION_STRING");

SmsClient smsClient = new SmsClient(connectionString);
```

## <a name="send-a-11-sms-message"></a>Envío de un SMS de un remitente a un solo destinatario

Para enviar un SMS a un solo destinatario, llame a la función `Send` o `SendAsync` desde SmsClient. Agregue este código al final del método `Main` en **Program.cs**:

```csharp
SmsSendResult sendResult = smsClient.Send(
    from: "<from-phone-number>",
    to: "<to-phone-number>",
    message: "Hello World via SMS"
);

Console.WriteLine($"Sms id: {sendResult.MessageId}");
```
Debe reemplazar `<from-phone-number>` por un número de teléfono habilitado para SMS asociado al recurso de Communication Services y `<to-phone-number>` por el número de teléfono al que quiere enviar un mensaje.

> [!WARNING]
> Tenga en cuenta que los números de teléfono se deben proporcionar en formato estándar internacional E.164. (por ejemplo, +14255550123).

## <a name="send-a-1n-sms-message-with-options"></a>Envío de un SMS de un remitente a varios destinatarios con opciones
Para enviar un SMS a una lista de destinatarios, llame a la función `Send` o `SendAsync` desde SmsClient con una lista de números de teléfono de destinatarios. También puede incluir parámetros opcionales para especificar si el informe de entregas debe estar habilitado y establecer etiquetas personalizadas.

```csharp
Response<IReadOnlyList<SmsSendResult>> response = smsClient.Send(
    from: "<from-phone-number>",
    to: new string[] { "<to-phone-number-1>", "<to-phone-number-2>" },
    message: "Weekly Promotion!",
    options: new SmsSendOptions(enableDeliveryReport: true) // OPTIONAL
    {
        Tag = "marketing", // custom tags
    });

IEnumerable<SmsSendResult> results = response.Value;
foreach (SmsSendResult result in results)
{
    Console.WriteLine($"Sms id: {result.MessageId}");
    Console.WriteLine($"Send Result Successful: {result.Successful}");
}
```

Debe reemplazar `<from-phone-number>` por un número de teléfono habilitado para SMS asociado al recurso de Communication Services y `<to-phone-number-1>` y `<to-phone-number-2>` por los números de teléfono a los que quiere enviar un mensaje.

> [!WARNING]
> Tenga en cuenta que los números de teléfono se deben proporcionar en formato estándar internacional E.164. (por ejemplo, +14255550123).

El parámetro `enableDeliveryReport` es un parámetro opcional que puede usar para configurar los informes de entrega. Resulta útil para los escenarios en los que quiere emitir eventos cuando se entregan mensajes SMS. Consulte el inicio rápido [Control de eventos SMS](../handle-sms-events.md) a fin de configurar los informes de entrega para los mensajes SMS.

`Tag` se usa para aplicar una etiqueta al informe de entrega.

## <a name="run-the-code"></a>Ejecución del código

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```console
dotnet run
```

## <a name="sample-code"></a>Código de ejemplo

Puede descargar la aplicación de ejemplo de [GitHub](https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/SendSMS).
