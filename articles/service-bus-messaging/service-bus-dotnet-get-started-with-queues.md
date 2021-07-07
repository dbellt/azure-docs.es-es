---
title: Introducción a las colas de Azure Service Bus (Azure.Messaging.ServiceBus)
description: En este tutorial, creará una aplicación de consola de C# de .NET Core para enviar mensajes a una cola de Service Bus y recibir mensajes desde ella.
ms.topic: quickstart
ms.tgt_pltfrm: dotnet
ms.date: 06/10/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: d82ee765e61fc743473801cf14752013790b5578
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112020743"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-net"></a>Envío y recepción de mensajes con destino y origen en colas de Azure Service Bus (.NET)
En esta guía de inicio rápido se muestra cómo enviar y recibir mensajes a y de una cola de Service Bus mediante la biblioteca de .NET [Azure.Messaging.ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus/).


## <a name="prerequisites"></a>Prerrequisitos
Si no está familiarizado con el servicio, consulte la [información general sobre Service Bus](service-bus-messaging-overview.md) antes de seguir este artículo de inicio rápido. 

- **Suscripción de Azure**. Para usar los servicios de Azure, entre los que se incluye Azure Service Bus, se necesita una suscripción.  Si no se dispone de una cuenta de Azure, es posible registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/free/), o bien usar las ventajas que disfrutan los suscriptores MSDN al [crear una cuenta](https://azure.microsoft.com).
- **Microsoft Visual Studio 2019**. La biblioteca cliente de Azure Service Bus usa las nuevas características que se introdujeron en C# 8.0.  Aunque puede seguir usando la biblioteca con versiones anteriores de C#, la nueva sintaxis no estará disponible. Para usar la sintaxis completa, se recomienda realizar la compilación con el [SDK de .NET Core](https://dotnet.microsoft.com/download) 3.0 o posterior y la [versión de lenguaje](/dotnet/csharp/language-reference/configure-language-version#override-a-default) establecida en `latest`. Si usa Visual Studio, las versiones anteriores a Visual Studio 2019 no son compatibles con las herramientas necesarias para compilar proyectos de C# 8.0. Visual Studio 2019, incluida la edición gratuita Community, se puede descargar [aquí](https://visualstudio.microsoft.com/vs/).
- **Creación de un espacio de nombres de Service Bus y una cola**. Siga los pasos descritos en el artículo [Uso de Azure Portal para crear un espacio de nombres de Service Bus y una cola](service-bus-quickstart-portal.md) para crear una cola y un espacio de nombres de Service Bus. 

    > [!IMPORTANT]
    > Anote la **cadena de conexión** para el espacio de nombres de Service Bus y el nombre de la **cola** que creó. Los usará más adelante en este tutorial. 


## <a name="send-messages"></a>Envío de mensajes
En esta sección se muestra cómo crear una aplicación de consola de .NET Core para mensajes a una cola de Service Bus. 

### <a name="create-a-console-application"></a>Creación de una aplicación de consola

1. Inicie Visual Studio 2019. 
1. Seleccione **Crear un proyecto**. 
1. En el cuadro de diálogo **Crear un nuevo proyecto**, siga estos pasos: Si no ve este cuadro de diálogo, seleccione **Archivo** en el menú, seleccione **Nuevo** y, después, seleccione **Proyecto**. 
    1. Seleccione **C#** como lenguaje de programación.
    1. Seleccione **Consola** como tipo de aplicación. 
    1. Seleccione **Aplicación de consola** de la lista de resultados. 
    1. Después, seleccione **Siguiente**. 

        :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/new-send-project.png" alt-text="Imagen que muestra el cuadro de diálogo Crear un proyecto con C# y Consola seleccionados":::
1. Escriba **QueueSender** como nombre del proyecto, **ServiceBusQueueQuickStart** como nombre de la solución y, a continuación, seleccione **Siguiente**. 

    :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/project-solution-names.png" alt-text="Imagen que muestra los nombres de solución y proyecto en el cuadro de diálogo Configurar el nuevo proyecto ":::
1. En la página **Información adicional**, seleccione **Crear** para crear la solución y el proyecto. 

### <a name="add-the-service-bus-nuget-package"></a>Agregar el paquete NuGet de Service Bus

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes** en el menú. 
1. Ejecute el siguiente comando para instalar el paquete NuGet **Azure.Messaging.ServiceBus**:

    ```cmd
    Install-Package Azure.Messaging.ServiceBus
    ```

### <a name="add-code-to-send-messages-to-the-queue"></a>Incorporación de código para enviar mensajes a la cola

1. En **Program.cs**, agregue las siguientes instrucciones `using` a la parte superior del archivo después de la instrucción `using` actual. 

    ```csharp
    using System.Threading.Tasks;    
    using Azure.Messaging.ServiceBus;
    ```
1. En la clase `Program`, agregue las dos propiedades estáticas siguientes. 

    ```csharp
        // connection string to your Service Bus namespace
        static string connectionString = "<NAMESPACE CONNECTION STRING>";

        // name of your Service Bus queue
        static string queueName = "<QUEUE NAME>";
    ```

    > [!NOTE]
    > Reemplace `<NAMESPACE CONNECTION STRING>` por la cadena de conexión del espacio de nombres de Service Bus. Además, reemplace `<QUEUE NAME>` por el nombre de la cola. 
1. Declare las siguientes propiedades estáticas en la clase `Program`. Vea los comentarios de código para obtener más detalles. 

    ```csharp
        // the client that owns the connection and can be used to create senders and receivers
        static ServiceBusClient client;

        // the sender used to publish messages to the queue
        static ServiceBusSender sender;

        // number of messages to be sent to the queue
        private const int numOfMessages = 3; 
    ```
1. Reemplace el método `Main()` por el método `Main`**asincrónico**.  

    ```csharp
        static async Task Main()
        {
            // The Service Bus client types are safe to cache and use as a singleton for the lifetime
            // of the application, which is best practice when messages are being published or read
            // regularly.
            //
            // Create the clients that we'll use for sending and processing messages.
            client = new ServiceBusClient(connectionString);
            sender = client.CreateSender(queueName);

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
                // Use the producer client to send the batch of messages to the Service Bus queue
                await sender.SendMessagesAsync(messageBatch);
                Console.WriteLine($"A batch of {numOfMessages} messages has been published to the queue.");
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
    ```
1. Compile el proyecto y asegúrese de que no hay errores. 
1. Ejecute el programa y espere el mensaje de confirmación.
    
    ```bash
    A batch of 3 messages has been published to the queue
    ```
1. En Azure Portal, haga lo siguiente:
    1. Vaya al espacio de nombres de Service Bus. 
    1. En la página **Información general**, seleccione la cola en el panel inferior central. 
    
        :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/select-queue.png" alt-text="Imagen en la que se muestra Espacio de nombres de Service Bus en Azure Portal con la cola seleccionada" lightbox="./media/service-bus-dotnet-get-started-with-queues/select-queue.png":::.
    1. Observe los valores en la sección **Información esencial**.

    :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/sent-messages-essentials.png" alt-text="Imagen que muestra el número de mensajes recibidos y el tamaño de la cola" lightbox="./media/service-bus-dotnet-get-started-with-queues/sent-messages-essentials.png":::.

    Observe los valores siguientes:
    - El valor del recuento de mensajes **Activos** de la cola ahora es **3**. Cada vez que se ejecuta la aplicación de remitente sin recuperar los mensajes, este valor aumenta en 3.
    - El **tamaño actual** de la cola aumenta cada vez que la aplicación agrega mensajes a la misma.
    - En el gráfico **Mensajes** de la sección **Métricas** inferior, puede ver que hay tres mensajes entrantes para la cola. 

## <a name="receive-messages"></a>Recepción de mensajes
En esta sección, creará una aplicación de consola de .NET Core que recibe mensajes de la cola. 

### <a name="create-a-project-for-the-receiver"></a>Creación de un proyecto para el destinatario

1. En la ventana del Explorador de soluciones, haga clic con el botón derecho en la solución **ServiceBusQueueQuickStart**, haga clic en **Agregar** y seleccione **Nuevo proyecto**. 
1. Seleccione **Aplicación de consola** y elija **Siguiente**. 
1. Escriba **QueueReceiver** en **Nombre de proyecto** y seleccione **Crear**. 
1. En la ventana del **Explorador de soluciones**, haga clic con el botón derecho en **QueueReceiver** y seleccione **Set as a Startup Project** (Establecer como proyecto de inicio). 

### <a name="add-the-service-bus-nuget-package"></a>Agregar el paquete NuGet de Service Bus

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes** en el menú. 
1. En la ventana de la **Consola del Administrador de paquetes**, confirme que **QueueReceiver** está seleccionado como **Proyecto predeterminado**. Si no es así, use la lista desplegable para seleccionar **QueueReceiver**.
1. Ejecute el siguiente comando para instalar el paquete NuGet **Azure.Messaging.ServiceBus**:

    ```cmd
    Install-Package Azure.Messaging.ServiceBus
    ```

### <a name="add-the-code-to-receive-messages-from-the-queue"></a>Adición del código para recibir mensajes de la cola
En esta sección agregará código para recuperar mensajes de la cola.

1. En *Program.cs*, agregue las siguientes instrucciones `using` en la parte superior de la definición del espacio de nombres, antes de la declaración de clase:

    ```csharp
    using System.Threading.Tasks;
    using Azure.Messaging.ServiceBus;
    ```

1. En la clase `Program`, declare las siguientes propiedades estáticas:

    ```csharp
        // connection string to your Service Bus namespace
        static string connectionString = "<NAMESPACE CONNECTION STRING>";

        // name of your Service Bus queue
        static string queueName = "<QUEUE NAME>";
    ```

    > [!NOTE]
    > Reemplace `<NAMESPACE CONNECTION STRING>` por la cadena de conexión del espacio de nombres de Service Bus. Además, reemplace `<QUEUE NAME>` por el nombre de la cola. 
1. Declare las siguientes propiedades estáticas en la clase `Program`. Vea los comentarios de código para obtener más detalles. 

    ```csharp
        // the client that owns the connection and can be used to create senders and receivers
        static ServiceBusClient client;

        // the processor that reads and processes messages from the queue
        static ServiceBusProcessor processor;
    ```
1. Agregue los métodos siguientes a la clase `Program` que controlan los mensajes y los errores. 

    ```csharp
        // handle received messages
        static async Task MessageHandler(ProcessMessageEventArgs args)
        {
            string body = args.Message.Body.ToString();
            Console.WriteLine($"Received: {body}");

            // complete the message. messages is deleted from the queue. 
            await args.CompleteMessageAsync(args.Message);
        }

        // handle any errors when receiving messages
        static Task ErrorHandler(ProcessErrorEventArgs args)
        {
            Console.WriteLine(args.Exception.ToString());
            return Task.CompletedTask;
        }
    ```
1. Reemplace el método `Main()`. Llama al método `ReceiveMessages` para recibir mensajes de la cola. 

    ```csharp
        static async Task Main()
        {
            // The Service Bus client types are safe to cache and use as a singleton for the lifetime
            // of the application, which is best practice when messages are being published or read
            // regularly.
            //

            // Create the client object that will be used to create sender and receiver objects
            client = new ServiceBusClient(connectionString);

            // create a processor that we can use to process the messages
            processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions());

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
    ```
1. Compile el proyecto y asegúrese de que no hay errores.
1. Ejecute la aplicación del destinatario. Debería ver los mensajes recibidos. Presione cualquier tecla para detener el receptor y la aplicación. 

    ```console
    Wait for a minute and then press any key to end the processing
    Received: Message 1
    Received: Message 2
    Received: Message 3
    
    Stopping the receiver...
    Stopped receiving messages
    ```
1. Vuelva a consultar el portal. Espere unos minutos y actualice la página si no ve `0` para los mensajes **Activos**. 

    - El recuento de mensajes **Activos** y los valores de **Tamaño actual** ahora son **0**.
    - En el gráfico **Mensajes** de la sección **Métricas** inferior, puede ver que hay ocho mensajes entrantes y otros ocho salientes para la cola. 
    
        :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/queue-messages-size-final.png" alt-text="Mensajes activos y tamaño después de la recepción" lightbox="./media/service-bus-dotnet-get-started-with-queues/queue-messages-size-final.png":::


## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación y los ejemplos siguientes:

- [Biblioteca cliente de Azure Service Bus para .NET: léame](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus)
- [Ejemplos en GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus/samples)
- [Referencia de API de .NET](/dotnet/api/azure.messaging.servicebus?preserve-view=true)