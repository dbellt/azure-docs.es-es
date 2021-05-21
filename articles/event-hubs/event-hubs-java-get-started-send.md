---
title: Envío o recepción de eventos desde Azure Event Hubs mediante Java (más reciente)
description: Este artículo es un tutorial para crear una aplicación de Java que envíe eventos a Azure Event Hubs, o los reciba de él, mediante el paquete azure-messaging-eventhubs más reciente.
ms.topic: quickstart
ms.date: 04/30/2021
ms.custom: devx-track-java
ms.openlocfilehash: e5d9df512c2066603ec8dcbcb876da1969a43161
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108753000"
---
# <a name="use-java-to-send-events-to-or-receive-events-from-azure-event-hubs-azure-messaging-eventhubs"></a>Use Java para enviar eventos a Azure Event Hubs o recibir eventos de él (azure-messaging-eventhubs)

En este inicio rápido se muestra cómo enviar y recibir eventos desde un centro de eventos mediante el paquete de Java **azure-messaging-eventhubs**.

> [!IMPORTANT]
> Este inicio rápido usa el nuevo paquete **azure-messaging-eventhubs**. Para ver un inicio rápido que use los paquetes **azure-eventhubs** y **azure-eventhubs-eph** anteriores, consulte [Envío y recepción de eventos mediante azure-eventhubs y azure-eventhubs-eph](event-hubs-java-get-started-send-legacy.md).


## <a name="prerequisites"></a>Requisitos previos

Si es la primera vez que usa Azure Event Hubs, consulte la [información general de Event Hubs](event-hubs-about.md) antes de continuar con este inicio rápido.

Para completar este tutorial de inicio rápido, debe cumplir los siguientes requisitos previos:

- Una **suscripción a Microsoft Azure**. Para usar los servicios de Azure, entre los que se incluye Azure Event Hubs, se necesita una suscripción.  Si no se dispone de una cuenta de Azure, es posible registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/free/), o bien usar las ventajas que disfrutan los suscriptores MSDN al [crear una cuenta](https://azure.microsoft.com).
- Un entorno de desarrollo de Java. Este inicio rápido utiliza [Eclipse](https://www.eclipse.org/). Se requiere el kit de desarrollo de Java (JDK), versión 8 o posteriores.
- **Creación de un espacio de nombres de Event Hubs y un centro de eventos**. El primer paso consiste en usar [Azure Portal](https://portal.azure.com) para crear un espacio de nombres de tipo Event Hubs y obtener las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. Para crear un espacio de nombres y un centro de eventos, siga el procedimiento que se indica en [este artículo](event-hubs-create.md). Después, obtenga la **cadena de conexión para el espacio de nombres de Event Hubs**. Para ello, siga las instrucciones del artículo: [Obtenga la cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). La utilizará más adelante en este inicio rápido.

## <a name="send-events"></a>Envío de eventos

En esta sección se muestra cómo crear una aplicación de Java para enviar eventos a un centro de eventos.

### <a name="add-reference-to-azure-event-hubs-library"></a>Incorporación de una referencia a la biblioteca de Azure Event Hubs

La biblioteca de cliente de Java para Event Hubs está disponible en [repositorio central de Maven](https://search.maven.org/search?q=a:azure-messaging-eventhubs). Puede hacer referencia a esta biblioteca con la siguiente declaración de dependencia en el archivo de proyecto de Maven:

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-messaging-eventhubs</artifactId>
    <version>5.7.0</version>
</dependency>
```

> [!NOTE]
> Actualice la versión a la más reciente publicada en el repositorio Maven.

### <a name="write-code-to-send-messages-to-the-event-hub"></a>Escritura de código para enviar mensajes al centro de eventos

Para el ejemplo siguiente, primero cree un nuevo proyecto de Maven para una aplicación de consola o shell en su entorno de desarrollo de Java favorito. Agregue una clase denominada `Sender` y agréguele el código siguiente:

> [!IMPORTANT]
> Actualice `<Event Hubs namespace connection string>` con la cadena de conexión en el espacio de nombres de Event Hubs. Actualice `<Event hub name>` con el nombre del centro de eventos en el espacio de nombres.

```java
import com.azure.messaging.eventhubs.*;
import java.util.Arrays;
import java.util.List;

public class Sender {
    private static final String connectionString = "<Event Hubs namespace connection string>";
    private static final String eventHubName = "<Event hub name>";

    public static void main(String[] args) {
        publishEvents();
    }
}
```

### <a name="add-code-to-publish-events-to-the-event-hub"></a>Adición de código para publicar eventos en el centro de eventos

Agregue un método llamado `publishEvents` a la clase `Sender`:

```java
    /**
     * Code sample for publishing events.
     * @throws IllegalArgumentException if the EventData is bigger than the max batch size.
     */
    public static void publishEvents() {
        // create a producer client
        EventHubProducerClient producer = new EventHubClientBuilder()
            .connectionString(connectionString, eventHubName)
            .buildProducerClient();

        // sample events in an array
        List<EventData> allEvents = Arrays.asList(new EventData("Foo"), new EventData("Bar"));

        // create a batch
        EventDataBatch eventDataBatch = producer.createBatch();

        for (EventData eventData : allEvents) {
            // try to add the event from the array to the batch
            if (!eventDataBatch.tryAdd(eventData)) {
                // if the batch is full, send it and then create a new batch
                producer.send(eventDataBatch);
                eventDataBatch = producer.createBatch();

                // Try to add that event that couldn't fit before.
                if (!eventDataBatch.tryAdd(eventData)) {
                    throw new IllegalArgumentException("Event is too large for an empty batch. Max size: "
                        + eventDataBatch.getMaxSizeInBytes());
                }
            }
        }
        // send the last batch of remaining events
        if (eventDataBatch.getCount() > 0) {
            producer.send(eventDataBatch);
        }
        producer.close();
    }
```

Compile el programa y asegúrese de que no hay errores. Este programa se ejecutará después de ejecutar el programa del destinatario.

## <a name="receive-events"></a>Recepción de eventos

El código de este tutorial se basa en el [ejemplo EventProcessorClient de GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/EventProcessorBlobCheckpointStoreSample.java), que se puede examinar para ver toda la aplicación en funcionamiento.

> [!WARNING]
> Si ejecuta este código en Azure Stack Hub, experimentará errores en tiempo de ejecución a menos que tenga como destino una versión específica de la API de Storage. Esto se debe a que el SDK de Event Hubs usa la API de Azure Storage más reciente disponible en Azure, que puede que no esté disponible en la plataforma de Azure Stack Hub. Azure Stack Hub puede admitir una versión del SDK de Azure Blob Storage que no sea ninguna de las que suelen estar disponibles en Azure. Si usa Azure Blob Storage como almacén de puntos de control, compruebe la [versión de la API de Azure Storage admitida para la compilación de Azure Stack Hub](/azure-stack/user/azure-stack-acs-differences?#api-version) y establezca esa versión como destino en el código.
>
> Por ejemplo, si trabaja en la versión 2005 de Azure Stack Hub, la versión más reciente disponible para el servicio Storage es la 2019-02-02. De forma predeterminada, la biblioteca de cliente del SDK de Event Hubs usa la versión más reciente disponible en Azure (2019-07-07 en el momento de la versión del SDK). En este caso, además de seguir los pasos de esta sección, también tendrá que agregar código para usar como destino la versión 2019-02-02 de la API del servicio de almacenamiento. Consulte [este ejemplo en GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/EventProcessorWithCustomStorageVersion.java) para ver cómo usar como destino una versión específica de la API de Storage.


### <a name="create-an-azure-storage-and-a-blob-container"></a>Creación de una instancia de Azure Storage y un contenedor de blobs de Azure Storage

En este inicio rápido, se usa Azure Storage (concretamente Blob Storage) como almacén de puntos de control. La creación de puntos de control es un proceso por el que un procesador de eventos marca o confirma la posición del último evento procesado correctamente en una partición. Normalmente, el marcado de un punto de control se realiza en la función que procesa los eventos. Para obtener más información acerca de la creación de puntos de control, consulte la sección sobre el [procesador de eventos](event-processor-balance-partition-load.md).

Siga estos pasos para crear una cuenta de Azure Storage.

1. [Creación de una cuenta de Azure Storage](../storage/common/storage-account-create.md?tabs=azure-portal)
2. [Creación de un contenedor de blobs](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)
3. [Obtención de la cadena de conexión para una cuenta de almacenamiento](../storage/common/storage-configure-connection-string.md)

    Anote la **cadena de conexión** y el **nombre del contenedor**. Los usará en el código de recepción.

### <a name="add-event-hubs-libraries-to-your-java-project"></a>Incorporación de bibliotecas de Event Hubs al proyecto de Java

Agregue las siguientes dependencias en el archivo pom.xml.

- [azure-messaging-eventhubs](https://search.maven.org/search?q=a:azure-messaging-eventhubs)
- [azure-messaging-eventhubs-checkpointstore-blob](https://search.maven.org/search?q=a:azure-messaging-eventhubs-checkpointstore-blob)

```xml
<dependencies>
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-messaging-eventhubs</artifactId>
        <version>5.7.0</version>
    </dependency>
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-messaging-eventhubs-checkpointstore-blob</artifactId>
        <version>1.6.0</version>
    </dependency>
</dependencies>
```

1. Agregue las siguientes instrucciones **import** en la parte superior del archivo Java.

    ```java
    import com.azure.messaging.eventhubs.*;
    import com.azure.messaging.eventhubs.checkpointstore.blob.BlobCheckpointStore;
    import com.azure.messaging.eventhubs.models.*;
    import com.azure.storage.blob.*;
    import java.util.function.Consumer;
    ```
2. Cree una clase denominada `Receiver` y agréguele las siguientes variables de cadena. Reemplace los marcadores de posición por los valores correctos.

    > [!IMPORTANT]
    > Reemplace los marcadores de posición por los valores correctos.
    > - `<Event Hubs namespace connection string>` con la cadena de conexión en el espacio de nombres de Event Hubs. Actualice
    > - `<Event hub name>` con el nombre del centro de eventos en el espacio de nombres.
    > - `<Storage connection string>` por la cadena de conexión de su cuenta de Azure Storage.
    > - `<Storage container name>` con el nombre del contenedor de Azure Blob Storage.

    ```java
    private static final String connectionString = "<Event Hubs namespace connection string>";
    private static final String eventHubName = "<Event hub name>";
    private static final String storageConnectionString = "<Storage connection string>";
    private static final String storageContainerName = "<Storage container name>";
    ```
3. Agregue el siguiente método `main` a la clase.

    ```java
    public static void main(String[] args) throws Exception {
        // Create a blob container client that you use later to build an event processor client to receive and process events
        BlobContainerAsyncClient blobContainerAsyncClient = new BlobContainerClientBuilder()
            .connectionString(storageConnectionString)
            .containerName(storageContainerName)
            .buildAsyncClient();

        // Create a builder object that you will use later to build an event processor client to receive and process events and errors.
        EventProcessorClientBuilder eventProcessorClientBuilder = new EventProcessorClientBuilder()
            .connectionString(connectionString, eventHubName)
            .consumerGroup(EventHubClientBuilder.DEFAULT_CONSUMER_GROUP_NAME)
            .processEvent(PARTITION_PROCESSOR)
            .processError(ERROR_HANDLER)
            .checkpointStore(new BlobCheckpointStore(blobContainerAsyncClient));

        // Use the builder object to create an event processor client
        EventProcessorClient eventProcessorClient = eventProcessorClientBuilder.buildEventProcessorClient();

        System.out.println("Starting event processor");
        eventProcessorClient.start();

        System.out.println("Press enter to stop.");
        System.in.read();

        System.out.println("Stopping event processor");
        eventProcessorClient.stop();
        System.out.println("Event processor stopped.");

        System.out.println("Exiting process");
    }
    ```
4. Agregue los dos métodos auxiliares (`PARTITION_PROCESSOR` y `ERROR_HANDLER`) que procesan eventos y errores a la clase `Receiver`.

    ```java
    public static final Consumer<EventContext> PARTITION_PROCESSOR = eventContext -> {
        PartitionContext partitionContext = eventContext.getPartitionContext();
        EventData eventData = eventContext.getEventData();

        System.out.printf("Processing event from partition %s with sequence number %d with body: %s%n",
            partitionContext.getPartitionId(), eventData.getSequenceNumber(), eventData.getBodyAsString());

        // Every 10 events received, it will update the checkpoint stored in Azure Blob Storage.
        if (eventData.getSequenceNumber() % 10 == 0) {
            eventContext.updateCheckpoint();
        }
    };

    public static final Consumer<ErrorContext> ERROR_HANDLER = errorContext -> {
        System.out.printf("Error occurred in partition processor for partition %s, %s.%n",
            errorContext.getPartitionContext().getPartitionId(),
            errorContext.getThrowable());
    };
    ```

5. Compile el programa y asegúrese de que no hay errores.

## <a name="run-the-applications"></a>Ejecución de las aplicaciones

1. Ejecute primero la aplicación del **destinatario**.
1. Luego, ejecute la aplicación del **remitente**.
1. En la ventana de la aplicación del **destinatario**, confirme que ve los eventos publicados por la aplicación del remitente.

    ```cmd
    Starting event processor
    Press enter to stop.
    Processing event from partition 0 with sequence number 331 with body: Foo
    Processing event from partition 0 with sequence number 332 with body: Bar
    ```
    
1. Presione **ENTRAR** en la ventana de la aplicación del destinatario para detener la aplicación.

    ```cmd
    Starting event processor
    Press enter to stop.
    Processing event from partition 0 with sequence number 331 with body: Foo
    Processing event from partition 0 with sequence number 332 with body: Bar

    Stopping event processor
    Event processor stopped.
    Exiting process
    ```

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes ejemplos en GitHub:

- [Ejemplos de azure-messaging-eventhubs](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs/src/samples/java/com/azure/messaging/eventhubs)
- [Ejemplos de azure-messaging-eventhubs-checkpointstore-blob](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob)
