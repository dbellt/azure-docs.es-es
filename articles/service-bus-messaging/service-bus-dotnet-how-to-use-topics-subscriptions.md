---
title: Introducción a los temas y las suscripciones de Azure Service Bus
description: En este inicio rápido se muestra cómo enviar mensajes a temas de Azure Service Bus mediante el paquete azure-messaging-servicebus.
ms.topic: quickstart
ms.tgt_pltfrm: dotnet
ms.date: 06/29/2021
ms.custom: contperf-fy21q3
ms.openlocfilehash: 13fee0b418a295fb47c5eb44f12190c7655b5934
ms.sourcegitcommit: 8b38eff08c8743a095635a1765c9c44358340aa8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113092724"
---
# <a name="send-messages-to-an-azure-service-bus-topic-and-receive-messages-from-its-subscriptions-net"></a>Envío de mensajes a un tema de Azure Service Bus y recepción de mensajes de sus suscripciones (.NET)
En este artículo de inicio rápido se muestra cómo enviar mensajes a un tema de Service Bus y recibir mensajes de una suscripción a ese tema mediante la biblioteca [Azure.Messaging.ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus/) de .NET.

## <a name="prerequisites"></a>Prerrequisitos
Si no está familiarizado con el servicio, consulte la [información general sobre Service Bus](service-bus-messaging-overview.md) antes de seguir este artículo de inicio rápido. 

- **Suscripción de Azure**. Para usar los servicios de Azure, entre los que se incluye Azure Service Bus, se necesita una suscripción.  Si no se dispone de una cuenta de Azure, es posible registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/free/), o bien usar las ventajas que disfrutan los suscriptores MSDN al [crear una cuenta](https://azure.microsoft.com).
- **Microsoft Visual Studio 2019**. La biblioteca cliente de Azure Service Bus usa las nuevas características que se introdujeron en C# 8.0.  Aunque puede seguir usando la biblioteca con versiones anteriores de C#, la nueva sintaxis no estará disponible. Para usar la sintaxis completa, se recomienda realizar la compilación con el [SDK de .NET Core](https://dotnet.microsoft.com/download) 3.0 o superior y la [versión de lenguaje](/dotnet/csharp/language-reference/configure-language-version#override-a-default) establecida en `latest`. Si usa Visual Studio, las versiones anteriores a Visual Studio 2019 no son compatibles con las herramientas necesarias para compilar proyectos de C# 8.0. Visual Studio 2019, incluida la edición gratuita Community, se puede descargar [aquí](https://visualstudio.microsoft.com/vs/).
- Siga los pasos de este [Inicio rápido](service-bus-quickstart-topics-subscriptions-portal.md) para crear un tema de Service Bus y suscripciones al tema. 

    > [!IMPORTANT]
    > Anote la cadena de conexión al espacio de nombres, el nombre del tema y el nombre de una de las suscripciones al tema. Los usará más adelante en este tutorial.
 
## <a name="send-messages-to-a-topic"></a>Envío de mensajes a un tema
En esta sección se muestra cómo crear una aplicación de consola de .NET Core para mensajes a un tema de Service Bus. 

### <a name="create-a-console-application"></a>Creación de una aplicación de consola

1. Inicie Visual Studio 2019. 
1. Seleccione **Crear un proyecto**. 
1. En el cuadro de diálogo **Crear un nuevo proyecto**, siga estos pasos: Si no ve este cuadro de diálogo, seleccione **Archivo** en el menú, seleccione **Nuevo** y, después, seleccione **Proyecto**. 
    1. Seleccione **C#** como lenguaje de programación.
    1. Seleccione **Consola** como tipo de aplicación. 
    1. Seleccione **Aplicación de consola** de la lista de resultados. 
    1. Después, seleccione **Siguiente**. 

        :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/new-send-project.png" alt-text="Imagen que muestra el cuadro de diálogo Crear un nuevo proyecto con C# y Consola seleccionados":::
1. Escriba **TopicSender** como nombre del proyecto, **ServiceBusTopicQuickStart** como nombre de la solución y, a continuación, seleccione **Siguiente**. 
1. En la página **Información adicional**, seleccione **Crear** para crear la solución y el proyecto.

### <a name="add-the-service-bus-nuget-package"></a>Agregar el paquete NuGet de Service Bus

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes** en el menú. 
1. Ejecute el siguiente comando para instalar el paquete NuGet **Azure.Messaging.ServiceBus**:

    ```cmd
    Install-Package Azure.Messaging.ServiceBus
    ```

### <a name="add-code-to-send-messages-to-the-topic"></a>Incorporación de código para enviar mensajes al tema 

1. Reemplace el código del archivo **Program.cs** por el código siguiente. Estos son los pasos importantes del código.  
    1. Crea un objeto [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient) mediante la cadena de conexión al espacio de nombres. 
    1. Invoca al método [CreateSender](/dotnet/api/azure.messaging.servicebus.servicebusclient.createsender) en el objeto `ServiceBusClient` para crear un objeto [ServiceBusSender](/dotnet/api/azure.messaging.servicebus.servicebussender) para el tema específico de Service Bus.     
    1. Crea un objeto [ServiceBusMessageBatch](/dotnet/api/azure.messaging.servicebus.servicebusmessagebatch) mediante [ServiceBusSender.CreateMessageBatchAsync](/dotnet/api/azure.messaging.servicebus.servicebussender.createmessagebatchasync).
    1. Agrega mensajes al lote mediante [ServiceBusMessageBatch.TryAddMessage](/dotnet/api/azure.messaging.servicebus.servicebusmessagebatch.tryaddmessage). 
    1. Envía el lote de mensajes al tema de Service Bus mediante el método [ServiceBusSender.SendMessagesAsync](/dotnet/api/azure.messaging.servicebus.servicebussender.sendmessagesasync).
    
        Para más información, consulte los comentarios del código.
        ```csharp
        using System;
        using System.Threading.Tasks;
        using Azure.Messaging.ServiceBus;
        
        namespace TopicSender
        {
            class Program
            {
                // connection string to your Service Bus namespace
                static string connectionString = "<NAMESPACE CONNECTION STRING>";
        
                // name of your Service Bus topic
                static string topicName = "<TOPIC NAME>";
        
                // the client that owns the connection and can be used to create senders and receivers
                static ServiceBusClient client;
        
                // the sender used to publish messages to the topic
                static ServiceBusSender sender;
        
                // number of messages to be sent to the topic
                private const int numOfMessages = 3;
        
                static async Task Main()
                {
                    // The Service Bus client types are safe to cache and use as a singleton for the lifetime
                    // of the application, which is best practice when messages are being published or read
                    // regularly.
                    //
                    // Create the clients that we'll use for sending and processing messages.
                    client = new ServiceBusClient(connectionString);
                    sender = client.CreateSender(topicName);
        
                    // create a batch 
                    using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();
        
                    for (int i = 1; i <= 3; i++)
                    {
                        // try adding a message to the batch
                        if (!messageBatch.TryAddMessage(new ServiceBusMessage($"Message {i}")))
                        {
                            // if it is too large for the batch
                            throw new Exception($"The message {i} is too large to fit in the batch.");
                        }
                    }
        
                    try
                    {
                        // Use the producer client to send the batch of messages to the Service Bus topic
                        await sender.SendMessagesAsync(messageBatch);
                        Console.WriteLine($"A batch of {numOfMessages} messages has been published to the topic.");
                    }
                    finally
                    {
                        // Calling DisposeAsync on client types is required to ensure that network
                        // resources and other unmanaged objects are properly cleaned up.
                        await sender.DisposeAsync();
                        await client.DisposeAsync();
                    }
        
                    Console.WriteLine("Press any key to end the application");
                    Console.ReadKey();
                }
            }
        }    
        ```
1. Reemplace `<NAMESPACE CONNECTION STRING>` por la cadena de conexión del espacio de nombres de Service Bus. Y reemplace `<TOPIC NAME>` por el nombre del tema de Service Bus. 
1. Compile el proyecto y asegúrese de que no hay errores. 
1. Ejecute el programa y espere el mensaje de confirmación.
    
    ```bash
    A batch of 3 messages has been published to the topic
    ```
1. En Azure Portal, haga lo siguiente:
    1. Vaya al espacio de nombres de Service Bus. 
    1. En la página **Información general**, en el panel inferior central, cambie a la pestaña **Temas** y seleccione el tema de Service Bus. En el ejemplo siguiente, es `mytopic`.
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/select-topic.png" alt-text="Selección de tema":::
    1. En la página **Tema de Service Bus**, en el gráfico **Mensajes** de la sección **Métricas** inferior, puede ver que hay tres mensajes entrantes para la cola. Si no ve el valor, espere unos minutos y actualice la página para ver el gráfico actualizado. 

        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/sent-messages-essentials.png" alt-text="Mensajes enviados al tema" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/sent-messages-essentials.png":::
    4. Seleccione la suscripción en el panel inferior. En el ejemplo siguiente, es **S1**. En la página **Suscripción de Service Bus**, verá el **Recuento de mensajes activos** como **3**. La suscripción ha recibido los tres mensajes que ha enviado al tema, pero ningún receptor los ha recogido todavía. 
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page.png" alt-text="Mensajes recibidos en la suscripción" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page.png":::
    
## <a name="receive-messages-from-a-subscription"></a>Recepción de mensajes de una suscripción
En esta sección, creará una aplicación de consola de .NET Core que recibe mensajes de la suscripción al tema de Service Bus. 

### <a name="create-a-project-for-the-receiver"></a>Creación de un proyecto para el destinatario

1. En la ventana del Explorador de soluciones, haga clic con el botón derecho en la solución **ServiceBusTopicQuickStart**, haga clic en **Agregar** y seleccione **Nuevo proyecto**. 
1. Seleccione **Aplicación de consola** y seleccione **Siguiente**. 
1. Escriba **SubscriptionReceiver** como **Nombre del proyecto** y seleccione **Siguiente**. 
1. En la página **Información adicional**, seleccione **Crear**. 
1. En la ventana **Explorador de soluciones**, haga clic con el botón derecho en **SubscriptionReceiver** y seleccione **Establecer como proyecto de inicio**. 

### <a name="add-the-service-bus-nuget-package"></a>Agregar el paquete NuGet de Service Bus

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes** en el menú. 
1. En la ventana **Consola del Administrador de paquetes**, confirme que **SubscriptionReceiver** esté seleccionado para el **Proyecto predeterminado**. Si no es así, use la lista desplegable para seleccionar **SubscriptionReceiver**.
1. Ejecute el siguiente comando para instalar el paquete NuGet **Azure.Messaging.ServiceBus**:

    ```cmd
    Install-Package Azure.Messaging.ServiceBus
    ```

### <a name="add-code-to-receive-messages-from-the-subscription"></a>Adición de código para recibir mensajes de la suscripción
1. Reemplace el código del archivo **Program.cs** por el código siguiente. Estos son los pasos importantes del código.
    Estos son los pasos importantes del código:
    1. Crea un objeto [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient) mediante la cadena de conexión al espacio de nombres. 
    1. Invoca al método [CreateProcessor](/dotnet/api/azure.messaging.servicebus.servicebusclient.createprocessor) en el objeto `ServiceBusClient` para crear un objeto [ServiceBusProcessor](/dotnet/api/azure.messaging.servicebus.servicebusprocessor) para la cola de Service Bus especificada. 
    1. Especifica los controladores para los eventos [ProcessMessageAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.processmessageasync) y [ProcessErrorAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.processerrorasync) del objeto `ServiceBusProcessor`. 
    1. Inicia el procesamiento de mensajes; para ello, invoca el método [StartProcessingAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.startprocessingasync) en el objeto `ServiceBusProcessor`. 
    1. Cuando el usuario presiona una tecla para finalizar el procesamiento, invoca el método [StopProcessingAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.stopprocessingasync) en el objeto `ServiceBusProcessor`. 

        Para más información, consulte los comentarios del código.

        ```csharp
        using System;
        using System.Threading.Tasks;
        using Azure.Messaging.ServiceBus;
        
        namespace SubscriptionReceiver
        {
            class Program
            {
                // connection string to your Service Bus namespace
                static string connectionString = "<NAMESPACE CONNECTION STRING>";
        
                // name of the Service Bus topic
                static string topicName = "<SERVICE BUS TOPIC NAME>";
            
                // name of the subscription to the topic
                static string subscriptionName = "<SERVICE BUS - TOPIC SUBSCRIPTION NAME>";
        
                // the client that owns the connection and can be used to create senders and receivers
                static ServiceBusClient client;
        
                // the processor that reads and processes messages from the subscription
                static ServiceBusProcessor processor;
        
                // handle received messages
                static async Task MessageHandler(ProcessMessageEventArgs args)
                {
                    string body = args.Message.Body.ToString();
                    Console.WriteLine($"Received: {body} from subscription: {subscriptionName}");
        
                    // complete the message. messages is deleted from the subscription. 
                    await args.CompleteMessageAsync(args.Message);
                }
        
                // handle any errors when receiving messages
                static Task ErrorHandler(ProcessErrorEventArgs args)
                {
                    Console.WriteLine(args.Exception.ToString());
                    return Task.CompletedTask;
                }
        
                static async Task Main()
                {
                    // The Service Bus client types are safe to cache and use as a singleton for the lifetime
                    // of the application, which is best practice when messages are being published or read
                    // regularly.
                    //
                    // Create the clients that we'll use for sending and processing messages.
                    client = new ServiceBusClient(connectionString);
        
                    // create a processor that we can use to process the messages
                    processor = client.CreateProcessor(topicName, subscriptionName, new ServiceBusProcessorOptions());
        
                    try
                    {
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
                    finally
                    {
                        // Calling DisposeAsync on client types is required to ensure that network
                        // resources and other unmanaged objects are properly cleaned up.
                        await processor.DisposeAsync();
                        await client.DisposeAsync();
                    }
                }
            }
        }
        ```
1. Reemplace los marcadores de posición por los valores correctos:
    - `<NAMESPACE CONNECTION STRING>` por la cadena de conexión del espacio de nombres de Service Bus.
    - `<TOPIC NAME>` por el nombre del tema de Service Bus.
    - `<SERVICE BUS - TOPIC SUBSCRIPTION NAME>` por el nombre de la suscripción al tema. 
1. Compile el proyecto y asegúrese de que no hay errores.
1. Ejecute la aplicación del destinatario. Debería ver los mensajes recibidos. Presione cualquier tecla para detener el receptor y la aplicación. 

    ```console
    Wait for a minute and then press any key to end the processing
    Received: Message 1 from subscription: S1
    Received: Message 2 from subscription: S1
    Received: Message 3 from subscription: S1
    
    Stopping the receiver...
    Stopped receiving messages
    ```
1. Vuelva a consultar el portal. 
    - En la página **Tema de Service Bus**, en el gráfico **Mensajes**, podrá ver tres mensajes entrantes y tres mensajes salientes. Si no ve estos números, espere unos minutos y actualice la página para ver el gráfico actualizado. 
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/messages-size-final.png" alt-text="Mensajes enviados y recibidos" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/messages-size-final.png":::
    - En la página **Service Bus Subscription** (Suscripción de Service Bus), verá el **Recuento de mensajes activos** como cero. Esto se debe a que un receptor ha recibido mensajes de esta suscripción y ha completado los mensajes. 
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page-final.png" alt-text="Recuento de mensajes activos final en la suscripción" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page-final.png":::
        


## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación y los ejemplos siguientes:

- [Biblioteca cliente de Azure Service Bus para .NET: léame](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus)
- [Ejemplos de .NET para Azure Service Bus en GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus/samples)
- [Referencia de API de .NET](/dotnet/api/azure.messaging.servicebus?preserve-view=true)
