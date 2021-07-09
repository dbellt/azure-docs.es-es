---
title: Envío o recepción de eventos desde Azure Event Hubs mediante .NET (más reciente)
description: Este artículo es un tutorial para crear una aplicación de .NET Core que envíe eventos a Azure Event Hubs, o los reciba de él, mediante el paquete Microsoft.Azure.EventHubs más reciente.
ms.topic: quickstart
ms.date: 06/10/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: c6f28e46aff12b5730a1cc73f56fe9bd31805923
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112004385"
---
# <a name="send-events-to-and-receive-events-from-azure-event-hubs---net-azuremessagingeventhubs"></a>Envío y recepción de eventos desde Azure Event Hubs: .NET (Azure.Messaging.EventHubs) 
En este inicio rápido se muestra cómo enviar y recibir eventos desde un centro de eventos mediante la biblioteca de .NET **Azure.Messaging.EventHubs**. 

## <a name="prerequisites"></a>Prerrequisitos
Si es la primera vez que usa Azure Event Hubs, consulte la [información general de Event Hubs](event-hubs-about.md) antes de continuar con este inicio rápido. 

Para completar este tutorial de inicio rápido, debe cumplir los siguientes requisitos previos:

- Una **suscripción a Microsoft Azure**. Para usar los servicios de Azure, entre los que se incluye Azure Event Hubs, se necesita una suscripción.  Si no se dispone de una cuenta de Azure, es posible registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/free/), o bien usar las ventajas que disfrutan los suscriptores MSDN al [crear una cuenta](https://azure.microsoft.com).
- **Microsoft Visual Studio 2019**. La biblioteca cliente de Azure Event Hubs utiliza las nuevas características que se introdujeron en C# 8.0.  Aunque puede seguir usando la biblioteca con versiones anteriores de C#, la nueva sintaxis no estará disponible. Para usar la sintaxis completa, se recomienda realizar la compilación con el [SDK de .NET Core](https://dotnet.microsoft.com/download) 3.0 o superior y la [versión de lenguaje](/dotnet/csharp/language-reference/configure-language-version#override-a-default) establecida en `latest`. Si usa Visual Studio, las versiones anteriores a Visual Studio 2019 no son compatibles con las herramientas necesarias para compilar proyectos de C# 8.0. Visual Studio 2019, incluida la edición gratuita Community, se puede descargar [aquí](https://visualstudio.microsoft.com/vs/).
- **Creación de un espacio de nombres de Event Hubs y un centro de eventos**. El primer paso consiste en usar [Azure Portal](https://portal.azure.com) para crear un espacio de nombres de tipo Event Hubs y obtener las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. Para crear un espacio de nombres y un centro de eventos, siga el procedimiento que se indica en [este artículo](event-hubs-create.md). Después, obtenga la **cadena de conexión para el espacio de nombres de Event Hubs**. Para ello, siga las instrucciones del artículo: [Obtenga la cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). La utilizará más adelante en este inicio rápido.

## <a name="send-events"></a>Envío de eventos 
En esta sección se muestra cómo crear una aplicación de consola de .NET Core para enviar eventos a un centro de eventos. 

### <a name="create-a-console-application"></a>Creación de una aplicación de consola

1. Inicie Visual Studio 2019. 
1. Seleccione **Crear un proyecto**. 
1. En el cuadro de diálogo **Crear un nuevo proyecto**, siga estos pasos: Si no ve este cuadro de diálogo, seleccione **Archivo** en el menú, seleccione **Nuevo** y, después, seleccione **Proyecto**. 
    1. Seleccione **C#** como lenguaje de programación.
    1. Seleccione **Consola** como tipo de aplicación. 
    1. Seleccione **Aplicación de consola** en la lista de resultados. 
    1. Después, seleccione **Siguiente**. 

        :::image type="content" source="./media/getstarted-dotnet-standard-send-v2/new-send-project.png" alt-text="Imagen que muestra el cuadro de diálogo Nuevo proyecto":::
1. Escriba **EventHubsSender** como nombre del proyecto, **EventHubsQuickStart** como nombre de la solución y, después, seleccione **Aceptar** para crear el proyecto. 

    :::image type="content" source="./media/getstarted-dotnet-standard-send-v2/project-solution-names.png" alt-text="Imagen que muestra la página donde se escriben los nombres de la solución y del proyecto":::

### <a name="add-the-event-hubs-nuget-package"></a>Incorporación del paquete NuGet de Event Hubs

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes** en el menú. 
1. Ejecute el siguiente comando para instalar el paquete NuGet **Azure.Messaging.EventHubs**:

    ```cmd
    Install-Package Azure.Messaging.EventHubs
    ```


### <a name="write-code-to-send-events-to-the-event-hub"></a>Escritura de código para enviar eventos al centro de eventos

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs**:

    ```csharp
    using System;
    using System.Text;
    using System.Threading.Tasks;
    using Azure.Messaging.EventHubs;
    using Azure.Messaging.EventHubs.Producer;
    ```

2. Agregue constantes a la clase `Program` de la cadena de conexión de Event Hubs y del nombre del centro de eventos.  

    ```csharp
        // connection string to the Event Hubs namespace
        private const string connectionString = "<EVENT HUBS NAMESPACE - CONNECTION STRING>";

        // name of the event hub
        private const string eventHubName = "<EVENT HUB NAME>";

        // number of events to be sent to the event hub
        private const int numOfEvents = 3;
    ```

    > [!NOTE]
    > Reemplace los valores de marcador de posición por la cadena de conexión al espacio de nombres y el nombre del centro de eventos. Asegúrese de que la cadena de conexión es la cadena de conexión de nivel de espacio de nombres.
3. Agregue la siguiente propiedad estática a la clase `Program`. Vea los comentarios de código. 

    ```csharp
        // The Event Hubs client types are safe to cache and use as a singleton for the lifetime
        // of the application, which is best practice when events are being published or read regularly.
        static EventHubProducerClient producerClient;    
    ```
1. Reemplace el método `Main` por el siguiente método `async Main`. Vea los comentarios de código para obtener más detalles. 

    ```csharp
        static async Task Main()
        {
            // Create a producer client that you can use to send events to an event hub
            producerClient = new EventHubProducerClient(connectionString, eventHubName);

            // Create a batch of events 
            using EventDataBatch eventBatch = await producerClient.CreateBatchAsync();

            for (int i = 1; i <= 3; i++)
            {
                if (! eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes($"Event {i}"))))
                {
                    // if it is too large for the batch
                    throw new Exception($"Event {i} is too large for the batch and cannot be sent.");
                }
            }

            try
            {
                // Use the producer client to send the batch of events to the event hub
                await producerClient.SendAsync(eventBatch);
                Console.WriteLine("A batch of 3 events has been published.");
            }
            finally
            {
                await producerClient.DisposeAsync();
            }
        }
    ```
5. Compile el proyecto y asegúrese de que no hay errores.
6. Ejecute el programa y espere el mensaje de confirmación. 

    ```csharp
    A batch of 3 events has been published.
    ```
1. En Azure Portal, puede comprobar que el centro de eventos ha recibido los eventos. Cambie a la vista **Mensajes** en la sección **Métricas**. Actualice la página para actualizar el gráfico. Puede tardar unos segundos en mostrar que los mensajes se han recibido. 

    :::image type="content" source="./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png" alt-text="Imagen de la página de Azure Portal para comprobar que el centro de eventos recibió los eventos" lightbox="./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png":::

    > [!NOTE]
    > Para ver el código fuente completo con comentarios muy útiles, consulte [este archivo en GitHub](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/samples/Sample04_PublishingEvents.md).

## <a name="receive-events"></a>Recepción de eventos
En esta sección se muestra cómo escribir una aplicación de consola de .NET Core que reciba eventos de un centro de eventos mediante un procesador de eventos. El procesador de eventos simplifica la recepción de eventos desde centros de eventos mediante la administración de puntos de control persistentes y las recepciones paralelas desde esos centros de eventos. Un procesador de eventos está asociado a un centro de eventos concreto y a un grupo de consumidores. Recibe eventos de varias particiones del centro de eventos y los pasa a un controlador delegado para que realice el procesamiento mediante el código que se ha especificado. 


> [!WARNING]
> Si ejecuta este código en **Azure Stack Hub**, experimentará errores en tiempo de ejecución a menos que tenga como destino una versión específica de la API de Storage. Esto se debe a que el SDK de Event Hubs usa la API de Azure Storage más reciente disponible en Azure, que puede que no esté disponible en la plataforma de Azure Stack Hub. Azure Stack Hub puede admitir una versión diferente del SDK de Blob Storage que las que suelen estar disponibles en Azure. Si usa Azure Blob Storage como almacén de puntos de control, compruebe la [versión de la API de Azure Storage admitida para la compilación de Azure Stack Hub](/azure-stack/user/azure-stack-acs-differences?#api-version) y establezca esa versión como destino en el código. 
>
> Por ejemplo, si trabaja en la versión 2005 de Azure Stack Hub, la versión más reciente disponible para el servicio Storage es la 2019-02-02. De forma predeterminada, la biblioteca de cliente del SDK de Event Hubs usa la versión más reciente disponible en Azure (2019-07-07 en el momento de la versión del SDK). En este caso, además de seguir los pasos de esta sección, también tendrá que agregar código para usar como destino la versión 2019-02-02 de la API del servicio de almacenamiento. Consulte [este ejemplo en GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor/samples/) para ver cómo usar como destino una versión específica de la API de Storage. 
 

### <a name="create-an-azure-storage-and-a-blob-container"></a>Creación de una instancia de Azure Storage y un contenedor de blobs de Azure Storage
En este inicio rápido, se usa Azure Storage como almacén de puntos de control. Siga estos pasos para crear una cuenta de Azure Storage. 

1. [Creación de una cuenta de Azure Storage](../storage/common/storage-account-create.md?tabs=azure-portal)
2. [Creación de un contenedor de blobs](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)
3. [Obtención de la cadena de conexión para una cuenta de almacenamiento](../storage/common/storage-configure-connection-string.md)

    Anote la cadena de conexión y el nombre del contenedor. Los usará en el código de recepción. 


### <a name="create-a-project-for-the-receiver"></a>Creación de un proyecto para el destinatario

1. En la ventana del Explorador de soluciones, haga clic con el botón derecho en la solución **EventHubQuickStart**, haga clic en **Agregar** y seleccione **Nuevo proyecto**. 
1. Seleccione **Aplicación de consola** y elija **Siguiente**. 
1. Escriba **EventHubsReceiver** en **Nombre de proyecto** y seleccione **Crear**. 
1. En la ventana del **Explorador de soluciones**, haga clic con el botón derecho en **EventHubsReceiver** y seleccione **Set as a Startup Project** (Establecer como proyecto de inicio). 

### <a name="add-the-event-hubs-nuget-package"></a>Incorporación del paquete NuGet de Event Hubs

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes** en el menú. 
1. En la ventana de la **Consola del Administrador de paquetes**, confirme que **EventHubsReceiver** está seleccionado como **Proyecto predeterminado**. Si no es así, use la lista desplegable para seleccionarlo.
1. Ejecute el siguiente comando para instalar el paquete NuGet **Azure.Messaging.EventHubs**:

    ```cmd
    Install-Package Azure.Messaging.EventHubs
    ```
1. Ejecute el siguiente comando para instalar el paquete NuGet **Azure.Messaging.EventHubs.Processor**:

    ```cmd
    Install-Package Azure.Messaging.EventHubs.Processor
    ```    

### <a name="update-the-main-method"></a>Actualización del método Main 

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs**.

    ```csharp
    using System;
    using System.Text;
    using System.Threading.Tasks;
    using Azure.Storage.Blobs;
    using Azure.Messaging.EventHubs;
    using Azure.Messaging.EventHubs.Consumer;
    using Azure.Messaging.EventHubs.Processor;
    ```
1. Agregue constantes a la clase `Program` de la cadena de conexión de Event Hubs y del nombre del centro de eventos. Reemplace los marcadores de posición entre llaves por los valores adecuados que recibió al crear el centro de eventos. Reemplace los marcadores de posición entre llaves por los valores adecuados que recibió al crear el centro de eventos y la cuenta de almacenamiento (claves de acceso y cadena de conexión principal). Asegúrese de que `{Event Hubs namespace connection string}` es la cadena de conexión en el nivel de espacio de nombres y no la cadena de Event Hub.

    ```csharp
        private const string ehubNamespaceConnectionString = "<EVENT HUBS NAMESPACE - CONNECTION STRING>";
        private const string eventHubName = "<EVENT HUB NAME>";
        private const string blobStorageConnectionString = "<AZURE STORAGE CONNECTION STRING>";
        private const string blobContainerName = "<BLOB CONTAINER NAME>";
    ```
3. Agregue las siguientes propiedades estáticas a la clase `Program`. 

    ```csharp
        static BlobContainerClient storageClient;

        // The Event Hubs client types are safe to cache and use as a singleton for the lifetime
        // of the application, which is best practice when events are being published or read regularly.        
        static EventProcessorClient processor;    
    ```
1. Reemplace el método `Main` por el siguiente método `async Main`. Vea los comentarios de código para obtener más detalles. 

    ```csharp
        static async Task Main()
        {
            // Read from the default consumer group: $Default
            string consumerGroup = EventHubConsumerClient.DefaultConsumerGroupName;

            // Create a blob container client that the event processor will use 
            storageClient = new BlobContainerClient(blobStorageConnectionString, blobContainerName);

            // Create an event processor client to process events in the event hub
            processor = new EventProcessorClient(storageClient, consumerGroup, ehubNamespaceConnectionString, eventHubName);

            // Register handlers for processing events and handling errors
            processor.ProcessEventAsync += ProcessEventHandler;
            processor.ProcessErrorAsync += ProcessErrorHandler;

            // Start the processing
            await processor.StartProcessingAsync();

            // Wait for 30 seconds for the events to be processed
            await Task.Delay(TimeSpan.FromSeconds(30));

            // Stop the processing
            await processor.StopProcessingAsync();
        }
    ```
1. Ahora, agregue los siguientes métodos de control de eventos y errores a la clase. 

    ```csharp
        static async Task ProcessEventHandler(ProcessEventArgs eventArgs)
        {
            // Write the body of the event to the console window
            Console.WriteLine("\tReceived event: {0}", Encoding.UTF8.GetString(eventArgs.Data.Body.ToArray()));

            // Update checkpoint in the blob storage so that the app receives only new events the next time it's run
            await eventArgs.UpdateCheckpointAsync(eventArgs.CancellationToken);
        }

        static Task ProcessErrorHandler(ProcessErrorEventArgs eventArgs)
        {
            // Write details about the error to the console window
            Console.WriteLine($"\tPartition '{ eventArgs.PartitionId}': an unhandled exception was encountered. This was not expected to happen.");
            Console.WriteLine(eventArgs.Exception.Message);
            return Task.CompletedTask;
        }    
    ```
1. Compile el proyecto y asegúrese de que no hay errores.

    > [!NOTE]
    > Para ver el código fuente completo con comentarios muy útiles, consulte [este archivo en GitHub](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor/samples/Sample01_HelloWorld.md).
6. Ejecute la aplicación del destinatario. 
1. Debería ver un mensaje que indica que se han recibido los eventos. 

    ```bash
    Received event: Event 1
    Received event: Event 2
    Received event: Event 3    
    ```
    Estos eventos son los tres que envió al centro de eventos anteriormente mediante la ejecución del programa del emisor. 


## <a name="next-steps"></a>Pasos siguientes
Consulte los ejemplos de GitHub. 

- [Ejemplos de Event Hubs en GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs/samples)
- [Ejemplos de procesador de eventos en GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor/samples)
- [Ejemplo del control de acceso basado en roles de Azure (Azure RBAC)](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Azure.Messaging.EventHubs/ManagedIdentityWebApp)
