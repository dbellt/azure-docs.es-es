---
title: Envío de mensajes a temas de Azure Service Bus mediante azure-messaging-servicebus
description: En este inicio rápido se muestra cómo enviar mensajes a temas de Azure Service Bus mediante el paquete azure-messaging-servicebus.
ms.topic: quickstart
ms.tgt_pltfrm: dotnet
ms.date: 03/16/2021
ms.custom: contperf-fy21q3
ms.openlocfilehash: 7cc854c850b02674151da7c88d94a2800b6572f5
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108015976"
---
# <a name="send-messages-to-an-azure-service-bus-topic-and-receive-messages-from-subscriptions-to-the-topic-net"></a>Envío de mensajes a un tema de Azure Service Bus y recepción de mensajes de las suscripciones a ese tema (.NET)
En este tutorial va a crear una aplicación de C# para realizar las tareas siguientes:

1. Enviar mensajes a un tema de Service Bus 

    Un tema Service Bus proporciona un punto de conexión para que las aplicaciones emisoras envíen mensajes. Un tema puede tener una o varias suscripciones. Cada suscripción a un tema obtiene una copia del mensaje enviado al tema. Para más información sobre los temas de Service Bus, consulte [Qué es Azure Service Bus](service-bus-messaging-overview.md). 
1. Recibir mensajes desde una suscripción al tema. 

    :::image type="content" source="./media/service-bus-messaging-overview/about-service-bus-topic.png" alt-text="Temas y suscripciones de Service Bus":::

    > [!Important]
    > Este inicio rápido usa el nuevo paquete **Azure.Messaging.ServiceBus**. Si utiliza el paquete Microsoft.Azure.ServiceBus anterior, consulte [Introducción a los temas de Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions-legacy.md).

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Para completar este tutorial, deberá tener una cuenta de Azure. Puede activar sus [beneficios de suscriptor de Visual Studio o MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Siga los pasos de este [Inicio rápido](service-bus-quickstart-topics-subscriptions-portal.md) para crear un tema de Service Bus y suscripciones al tema. 

    > [!NOTE]
    > En este tutorial, usará la cadena de conexión al espacio de nombres, el nombre del tema y el nombre de una de las suscripciones al tema.  
- [Visual Studio 2019](https://www.visualstudio.com/vs). 
 
## <a name="send-messages-to-a-topic"></a>Envío de mensajes a un tema
En esta sección creará una aplicación de consola de .NET Core en Visual Studio y agregará código para enviar mensajes al tema de Service Bus que ha creado. 

### <a name="create-a-console-application"></a>Creación de una aplicación de consola
Cree una aplicación de consola de .NET Core con Visual Studio. 

1. Inicie Visual Studio.  
1. Si ve la página **Comenzar**, seleccione **Crear un nuevo proyecto**. 
1. En la página **Crear un nuevo proyecto**, siga estos pasos: 
    1. Seleccione **C#** como lenguaje de programación. 
    1. Seleccione **Consola** como tipo de proyecto. 
    1. Seleccione **Aplicación de consola (.NET Core)** en la lista de plantillas. 
    1. Después, seleccione **Siguiente**. 
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/create-console-project.png" alt-text="Creación de un proyecto de aplicación de consola":::
1. En la página **Configurar el nuevo proyecto**, siga estos pasos: 
    1. En **Nombre del proyecto**, escriba un nombre para el proyecto. 
    1. En **Ubicación**, seleccione una ubicación para los archivos del proyecto y la solución. 
    1. En **Nombre de la solución**, escriba un nombre para la solución. Una solución de Visual Studio puede tener uno o varios proyectos. En esta guía de inicio rápido, la solución solo tendrá un proyecto. 
    1. Seleccione **Crear** para crear el proyecto. 
            
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/create-console-project-2.png" alt-text="Escribir el nombre y la ubicación de la solución y el proyecto":::    


### <a name="add-the-service-bus-nuget-package"></a>Agregar el paquete NuGet de Service Bus
1. Haga clic con el botón derecho en el proyecto recién creado y seleccione **Administrar paquetes NuGet**.

    :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/manage-nuget-packages-menu.png" alt-text="Administrar paquetes NuGet":::        
1. Cambie a la pestaña **Examinar**.
1. Busque y seleccione **[Azure.Messaging.ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus/)** .
1. Seleccione **Instalar** para completar la instalación.

    :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/select-service-bus-package.png" alt-text="Seleccionar el paquete NuGet de Service Bus.":::
5. Si ve el cuadro de diálogo **Vista previa de los cambios**, seleccione **Aceptar** para continuar. 
1. Si ve la página **Aceptación de la licencia**, seleccione **Acepto** para continuar. 
    

### <a name="add-code-to-send-messages-to-the-topic"></a>Incorporación de código para enviar mensajes al tema 

1. En la ventana **Explorador de soluciones**, haga doble clic en **Program.cs** para abrir el archivo en el editor. 
1. Agregue las siguientes instrucciones `using` en la parte superior de la definición del espacio de nombres, antes de la declaración de la clase:
   
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Azure.Messaging.ServiceBus;
    ```
1. En la clase `Program`, encima de la función `Main`, declare las variables siguientes:

    ```csharp
        static string connectionString = "<NAMESPACE CONNECTION STRING>";
        static string topicName = "<SERVICE BUS TOPIC NAME>";
        static string subscriptionName = "<SERVICE BUS - TOPIC SUBSCRIPTION NAME>";
    ```

    Reemplace los siguientes valores:
    - `<NAMESPACE CONNECTION STRING>` por la cadena de conexión del espacio de nombres de Service Bus.
    - `<TOPIC NAME>` por el nombre del tema.
    - `<SUBSCRIPTION NAME>` por el nombre de la suscripción.

### <a name="send-a-single-message-to-the-topic"></a>Envío de un único mensaje al tema
Agregue un método llamado `SendMessageToTopicAsync` a la clase `Program` encima o debajo del método `Main`. Este método envía un único mensaje al tema.

```csharp
    static async Task SendMessageToTopicAsync()
    {
        // create a Service Bus client 
        await using (ServiceBusClient client = new ServiceBusClient(connectionString))
        {
            // create a sender for the topic
            ServiceBusSender sender = client.CreateSender(topicName);
            await sender.SendMessageAsync(new ServiceBusMessage("Hello, World!"));
            Console.WriteLine($"Sent a single message to the topic: {topicName}");
        }
    }
```

Este método realiza los pasos siguientes: 
1. Crea un objeto [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient) mediante la cadena de conexión al espacio de nombres. 
1. Utiliza el objeto `ServiceBusClient` para crear un objeto [ServiceBusSender](/dotnet/api/azure.messaging.servicebus.servicebussender) para el tema de Service Bus especificado. En este paso se usa el método [ServiceBusClient.CreateSender](/dotnet/api/azure.messaging.servicebus.servicebusclient.createsender).
1. A continuación, el método envía un único mensaje de prueba al tema de Service Bus mediante el método [ServiceBusSender.SendMessageAsync](/dotnet/api/azure.messaging.servicebus.servicebussender.sendmessageasync). 

### <a name="send-a-batch-of-messages-to-the-topic"></a>Envío de un lote de mensajes al tema
1. Agregue un método llamado `CreateMessages` para crear una cola (una cola de .NET, no una cola de Service Bus) de mensajes a la clase `Program`. Normalmente, estos mensajes se obtienen de distintas partes de la aplicación. En este caso crearemos una cola de mensajes de ejemplo. Si no está familiarizado con las colas de .NET, consulte [Queue.Enqueue](/dotnet/api/system.collections.queue.enqueue).

    ```csharp
        static Queue<ServiceBusMessage> CreateMessages()
        {
            // create a queue containing the messages and return it to the caller
            Queue<ServiceBusMessage> messages = new Queue<ServiceBusMessage>();
            messages.Enqueue(new ServiceBusMessage("First message"));
            messages.Enqueue(new ServiceBusMessage("Second message"));
            messages.Enqueue(new ServiceBusMessage("Third message"));
            return messages;
        }
    ```
1. Agregue un método llamado `SendMessageBatchAsync` a la clase `Program` y agregue el código siguiente. Este método toma una cola de mensajes y prepara uno o varios lotes para enviarlos al tema de Service Bus. 

    ```csharp
        static async Task SendMessageBatchToTopicAsync()
        {
            // create a Service Bus client 
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {

                // create a sender for the topic 
                ServiceBusSender sender = client.CreateSender(topicName);

                // get the messages to be sent to the Service Bus topic
                Queue<ServiceBusMessage> messages = CreateMessages();

                // total number of messages to be sent to the Service Bus topic
                int messageCount = messages.Count;

                // while all messages are not sent to the Service Bus topic
                while (messages.Count > 0)
                {
                    // start a new batch 
                    using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();

                    // add the first message to the batch
                    if (messageBatch.TryAddMessage(messages.Peek()))
                    {
                        // dequeue the message from the .NET queue once the message is added to the batch
                        messages.Dequeue();
                    }
                    else
                    {
                        // if the first message can't fit, then it is too large for the batch
                        throw new Exception($"Message {messageCount - messages.Count} is too large and cannot be sent.");
                    }

                    // add as many messages as possible to the current batch
                    while (messages.Count > 0 && messageBatch.TryAddMessage(messages.Peek()))
                    {
                        // dequeue the message from the .NET queue as it has been added to the batch
                        messages.Dequeue();
                    }

                    // now, send the batch
                    await sender.SendMessagesAsync(messageBatch);

                    // if there are any remaining messages in the .NET queue, the while loop repeats 
                }

                Console.WriteLine($"Sent a batch of {messageCount} messages to the topic: {topicName}");
            }
        }
    ```    

    Estos son los pasos importantes del código:
    1. Crea un objeto [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient) mediante la cadena de conexión al espacio de nombres. 
    1. Invoca el método [CreateSender](/dotnet/api/azure.messaging.servicebus.servicebusclient.createsender) en el objeto `ServiceBusClient` para crear un objeto [ServiceBusSender](/dotnet/api/azure.messaging.servicebus.servicebussender) para el tema de Service Bus especificado. 
    1. Invoca el método auxiliar `CreateMessages` para obtener una cola de mensajes que se va a enviar al tema de Service Bus. 
    1. Crea un objeto [ServiceBusMessageBatch](/dotnet/api/azure.messaging.servicebus.servicebusmessagebatch) con [ServiceBusSender.CreateMessageBatchAsync](/dotnet/api/azure.messaging.servicebus.servicebussender.createmessagebatchasync).
    1. Agrega mensajes al lote mediante [ServiceBusMessageBatch.TryAddMessage](/dotnet/api/azure.messaging.servicebus.servicebusmessagebatch.tryaddmessage). A medida que los mensajes se agregan al lote, se quitan de la cola de .NET. 
    1. Envía el lote de mensajes al tema de Service Bus mediante el método [ServiceBusSender.SendMessagesAsync](/dotnet/api/azure.messaging.servicebus.servicebussender.sendmessagesasync).

### <a name="update-the-main-method"></a>Actualización del método main
Reemplace el método `Main()` por el método `Main`**asincrónico**. Llama a los métodos de envío para enviar un único mensaje y un lote de mensajes al tema.  

```csharp
    static async Task Main()
    {
        // send a single message to the topic
        await SendMessageToTopicAsync();

        // send a batch of messages to the topic
        await SendMessageBatchToTopicAsync();
    }
```

### <a name="test-the-app-to-send-messages-to-the-topic"></a>Prueba de la aplicación para enviar mensajes al tema
1. Ejecute la aplicación. Debería ver la siguiente salida:

    ```console
    Sent a single message to the topic: mytopic
    Sent a batch of 3 messages to the topic: mytopic
    ```
1. En Azure Portal, haga lo siguiente:
    1. Vaya al espacio de nombres de Service Bus. 
    1. En la página **Información general**, en el panel inferior central, cambie a la pestaña **Temas** y seleccione el tema de Service Bus. En el ejemplo siguiente, es `mytopic`.
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/select-topic.png" alt-text="Selección de tema":::
    1. En la página **Tema de Service Bus**, en el gráfico **Mensajes** de la sección **Métricas** inferior, puede ver que hay cuatro mensajes entrantes para la cola. Si no ve el valor, espere unos minutos y actualice la página para ver el gráfico actualizado. 

        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/sent-messages-essentials.png" alt-text="Mensajes enviados al tema" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/sent-messages-essentials.png":::
    4. Seleccione la suscripción en el panel inferior. En el ejemplo siguiente, es **S1**. En la página **Service Bus Subscription** (Suscripción de Service Bus), verá el **Recuento de mensajes activos** como **4**. La suscripción ha recibido los cuatro mensajes que ha enviado al tema, pero ningún receptor los ha recogido todavía. 
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page.png" alt-text="Mensajes recibidos en la suscripción" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page.png":::
    

## <a name="receive-messages-from-a-subscription"></a>Recepción de mensajes de una suscripción

1. Agregue los métodos siguientes a la clase `Program` que controlan los mensajes y los errores. 

    ```csharp
        static async Task MessageHandler(ProcessMessageEventArgs args)
        {
            string body = args.Message.Body.ToString();
            Console.WriteLine($"Received: {body} from subscription: {subscriptionName}");

            // complete the message. messages is deleted from the queue. 
            await args.CompleteMessageAsync(args.Message);
        }

        static Task ErrorHandler(ProcessErrorEventArgs args)
        {
            Console.WriteLine(args.Exception.ToString());
            return Task.CompletedTask;
        }
    ```
1. Agregue el siguiente método `ReceiveMessagesFromSubscriptionAsync` a la clase `Program`.

    ```csharp
        static async Task ReceiveMessagesFromSubscriptionAsync()
        {
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a processor that we can use to process the messages
                ServiceBusProcessor processor = client.CreateProcessor(topicName, subscriptionName, new ServiceBusProcessorOptions());

                // add handler to process messages
                processor.ProcessMessageAsync += MessageHandler;

                // add handler to process any errors
                processor.ProcessErrorAsync += ErrorHandler;

                // start processing 
                await processor.StartProcessingAsync();

                Console.WriteLine("Wait for a minute and then press any key to end the processing");
                Console.ReadKey();

                // stop processing 
                Console.WriteLine("\nStopping the receiver...");
                await processor.StopProcessingAsync();
                Console.WriteLine("Stopped receiving messages");
            }
        }
    ```

    Estos son los pasos importantes del código:
    1. Crea un objeto [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient) mediante la cadena de conexión al espacio de nombres. 
    1. Invoca el método [CreateProcessor](/dotnet/api/azure.messaging.servicebus.servicebusclient.createprocessor) en el objeto `ServiceBusClient` para crear un objeto [ServiceBusProcessor](/dotnet/api/azure.messaging.servicebus.servicebusprocessor) para el tema de Service Bus especificado y la combinación de suscripciones. 
    1. Especifica los controladores para los eventos [ProcessMessageAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.processmessageasync) y [ProcessErrorAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.processerrorasync) del objeto `ServiceBusProcessor`. 
    1. Inicia el procesamiento de mensajes; para ello, invoca el método [StartProcessingAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.startprocessingasync) en el objeto `ServiceBusProcessor`. 
    1. Cuando el usuario presiona una tecla para finalizar el procesamiento, invoca el método [StopProcessingAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.stopprocessingasync) en el objeto `ServiceBusProcessor`. 
1. Agregue una llamada al método `ReceiveMessagesFromSubscriptionAsync` al método `Main`. Convierta en comentario el método `SendMessagesToTopicAsync` si solo desea probar la recepción de mensajes. Si no lo hace, verá otros cuatro mensajes enviados al tema. 

    ```csharp
        static async Task Main()
        {
            // send a message to the topic
            await SendMessageToTopicAsync();

            // send a batch of messages to the topic
            await SendMessageBatchToTopicAsync();

            // receive messages from the subscription
            await ReceiveMessagesFromSubscriptionAsync();
        }
    ```
## <a name="run-the-app"></a>Ejecutar la aplicación
Ejecute la aplicación. Espere un minuto y, a continuación, presione cualquier tecla para dejar de recibir mensajes. Debería ver el resultado siguiente (barra espaciadora para la tecla). 

```console
Sent a single message to the topic: mytopic
Sent a batch of 3 messages to the topic: mytopic
Wait for a minute and then press any key to end the processing
Received: Hello, World! from subscription: mysub
Received: First message from subscription: mysub
Received: Second message from subscription: mysub
Received: Third message from subscription: mysub
Received: Hello, World! from subscription: mysub
Received: First message from subscription: mysub
Received: Second message from subscription: mysub
Received: Third message from subscription: mysub

Stopping the receiver...
Stopped receiving messages
```

Vuelva a consultar el portal. 

- En la página **Tema de Service Bus**, en el gráfico **Mensajes**, verá ocho mensajes entrantes y otros ocho salientes. Si no ve estos números, espere unos minutos y actualice la página para ver el gráfico actualizado. 

    :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/messages-size-final.png" alt-text="Mensajes enviados y recibidos" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/messages-size-final.png":::
- En la página **Service Bus Subscription** (Suscripción de Service Bus), verá el **Recuento de mensajes activos** como cero. Esto se debe a que un receptor ha recibido mensajes de esta suscripción y ha completado los mensajes. 

    :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page-final.png" alt-text="Recuento de mensajes activos final en la suscripción" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page-final.png":::
    


## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación y los ejemplos siguientes:

- [Biblioteca cliente de Azure Service Bus para .NET: léame](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus)
- [Ejemplos de .NET para Azure Service Bus en GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus/samples)
- [Referencia de API de .NET](/dotnet/api/azure.messaging.servicebus?preserve-view=true)
