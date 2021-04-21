---
title: Uso de las colas de Azure Service Bus con Java (azure-messaging-servicebus)
description: En este tutorial aprenderá a usar Java para enviar mensajes a una cola de Azure Service Bus y recibir mensajes desde allí. Para ello, se usa el nuevo paquete azure-messaging-servicebus.
ms.date: 02/13/2021
ms.topic: quickstart
ms.devlang: Java
ms.custom:
- seo-java-july2019
- seo-java-august2019
- seo-java-september2019
- devx-track-java
- mode-api
ms.openlocfilehash: e3998e812bb921ff8bea5e83199bc4e142ed2e83
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533418"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-java"></a>Envío de mensajes a las colas de Azure Service Bus (Java) y recepción desde ellas
En este inicio rápido creará aplicaciones de Java para enviar mensajes a una cola de Azure Service Bus y recibir mensajes desde allí. 

> [!IMPORTANT]
> En este inicio rápido se usa el nuevo paquete azure-messaging-servicebus. Para usar el paquete azure-servicebus anterior, consulte el inicio rápido [Envío y recepción de mensajes mediante azure-servicebus](service-bus-java-how-to-use-queues-legacy.md).


## <a name="prerequisites"></a>Requisitos previos
- Suscripción a Azure. Para completar este tutorial, deberá tener una cuenta de Azure. Puede activar sus [ventajas de suscriptor a MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Si no tiene una cola con la que trabajar, siga los pasos del artículo [Uso de Azure Portal para crear una cola de Service Bus](service-bus-quickstart-portal.md) para crear una. Anote la **cadena de conexión** para el espacio de nombres de Service Bus y el nombre de la **cola** que creó.
- Instale el [SDK de Azure para Java][Azure SDK for Java]. Si usa Eclipse, puede instalar [Azure Toolkit for Eclipse][Azure Toolkit for Eclipse], que incluye el SDK de Azure para Java. Después puede agregar las **Bibliotecas de Microsoft Azure para Java** al proyecto. Si utiliza IntelliJ, consulte [Instalación de Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/installation). 


## <a name="send-messages-to-a-queue"></a>mensajes a una cola
En esta sección creará un proyecto de consola de Java y agregará código para enviar mensajes a la cola que creó anteriormente. 

### <a name="create-a-java-console-project"></a>Creación de un proyecto de consola de Java
Cree un proyecto de Java mediante Eclipse o la herramienta que prefiera. 

### <a name="configure-your-application-to-use-service-bus"></a>Configuración de la aplicación para usar Service Bus
Agregue referencias a las bibliotecas de Azure Core y Azure Service Bus. 

Si usa Eclipse y ha creado una aplicación de consola de Java, convierta el proyecto de Java a Maven: haga clic con el botón derecho en el proyecto en la ventana del **Explorador de paquetes**, seleccione **Configurar** -> **Convert to Maven project** (Convertir en proyecto de Maven). Luego, agregue dependencias a estas dos bibliotecas, tal como se muestra en el ejemplo siguiente.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.myorg.sbusquickstarts</groupId>
    <artifactId>sbustopicqs</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <build>
        <sourceDirectory>src</sourceDirectory>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <release>15</release>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-core</artifactId>
            <version>1.13.0</version>
        </dependency>
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-messaging-servicebus</artifactId>
            <version>7.0.2</version>
        </dependency>
    </dependencies>
</project>
```

### <a name="add-code-to-send-messages-to-the-queue"></a>Incorporación de código para enviar mensajes a la cola
1. Agregue las siguientes instrucciones `import` al principio del archivo Java. 

    ```java
    import com.azure.messaging.servicebus.*;
    
    import java.util.concurrent.CountDownLatch;
    import java.util.concurrent.TimeUnit;
    import java.util.Arrays;
    import java.util.List;
    ```    
5. En la clase, defina las variables para almacenar la cadena de conexión y el nombre de la cola, como se muestra a continuación: 

    ```java
    static String connectionString = "<NAMESPACE CONNECTION STRING>";
    static String queueName = "<QUEUE NAME>";    
    ```

    Reemplace `<NAMESPACE CONNECTION STRING>` por la cadena de conexión del espacio de nombres de Service Bus. Reemplace `<QUEUE NAME>` por el nombre de la cola.
3. Agregue un método denominado `sendMessage` a la clase para enviar un mensaje a la cola. 

    ```java
    static void sendMessage()
    {
        // create a Service Bus Sender client for the queue 
        ServiceBusSenderClient senderClient = new ServiceBusClientBuilder()
                .connectionString(connectionString)
                .sender()
                .queueName(queueName)
                .buildClient();
        
        // send one message to the queue
        senderClient.sendMessage(new ServiceBusMessage("Hello, World!"));
        System.out.println("Sent a single message to the queue: " + queueName);        
    }
    ```
1. Agregue un método denominado `createMessages` a la clase para crear una lista de mensajes. Normalmente, estos mensajes se obtienen de distintas partes de la aplicación. En este caso crearemos una lista de mensajes de ejemplo.

    ```java
    static List<ServiceBusMessage> createMessages()
    {
        // create a list of messages and return it to the caller
        ServiceBusMessage[] messages = {
                new ServiceBusMessage("First message"),
                new ServiceBusMessage("Second message"),
                new ServiceBusMessage("Third message")
        };
        return Arrays.asList(messages);
    }
    ```
1. Agregue un método denominado `sendMessageBatch` para enviar mensajes a la cola que ha creado. Este método crea un `ServiceBusSenderClient` para la cola, invoca el método `createMessages` para obtener la lista de mensajes, prepara uno o varios lotes y envía los lotes a la cola. 

    ```java
    static void sendMessageBatch()
    {
        // create a Service Bus Sender client for the queue 
        ServiceBusSenderClient senderClient = new ServiceBusClientBuilder()
                .connectionString(connectionString)
                .sender()
                .queueName(queueName)
                .buildClient();

        // Creates an ServiceBusMessageBatch where the ServiceBus.
        ServiceBusMessageBatch messageBatch = senderClient.createMessageBatch();        
        
        // create a list of messages
        List<ServiceBusMessage> listOfMessages = createMessages();
        
        // We try to add as many messages as a batch can fit based on the maximum size and send to Service Bus when
        // the batch can hold no more messages. Create a new batch for next set of messages and repeat until all
        // messages are sent.        
        for (ServiceBusMessage message : listOfMessages) {
            if (messageBatch.tryAddMessage(message)) {
                continue;
            }

            // The batch is full, so we create a new batch and send the batch.
            senderClient.sendMessages(messageBatch);
            System.out.println("Sent a batch of messages to the queue: " + queueName);
            
            // create a new batch
            messageBatch = senderClient.createMessageBatch();

            // Add that message that we couldn't before.
            if (!messageBatch.tryAddMessage(message)) {
                System.err.printf("Message is too large for an empty batch. Skipping. Max size: %s.", messageBatch.getMaxSizeInBytes());
            }
        }
        
        if (messageBatch.getCount() > 0) {
            senderClient.sendMessages(messageBatch);
            System.out.println("Sent a batch of messages to the queue: " + queueName);
        }

        //close the client
        senderClient.close();
    }
    ```

## <a name="receive-messages-from-a-queue"></a>mensajes de una cola
En esta sección agregará código para recuperar mensajes de la cola. 

1. Agregue un método denominado `receiveMessages` para recibir mensajes de la cola. Este método crea un `ServiceBusProcessorClient` para la cola al especificar un controlador para procesar los mensajes y otro, para los errores. A continuación, inicia el procesador, espera unos segundos, imprime los mensajes que se reciben, y detiene y cierra el procesador.

    > [!IMPORTANT]
    > Reemplace `QueueTest` en `QueueTest::processMessage` en el código por el nombre de la clase. 

    ```java
    // handles received messages
    static void receiveMessages() throws InterruptedException
    {
        CountDownLatch countdownLatch = new CountDownLatch(1);

        // Create an instance of the processor through the ServiceBusClientBuilder
        ServiceBusProcessorClient processorClient = new ServiceBusClientBuilder()
            .connectionString(connectionString)
            .processor()
            .queueName(queueName)
            .processMessage(QueueTest::processMessage)
            .processError(context -> processError(context, countdownLatch))
            .buildProcessorClient();

        System.out.println("Starting the processor");
        processorClient.start();

        TimeUnit.SECONDS.sleep(10);
        System.out.println("Stopping and closing the processor");
        processorClient.close();        
    }   
    ```
2. Agregue el método `processMessage` para procesar un mensaje recibido de la suscripción de Service Bus. 

    ```java
    private static void processMessage(ServiceBusReceivedMessageContext context) {
        ServiceBusReceivedMessage message = context.getMessage();
        System.out.printf("Processing message. Session: %s, Sequence #: %s. Contents: %s%n", message.getMessageId(),
            message.getSequenceNumber(), message.getBody());
    }    
    ```
3. Agregue el método `processError` para controlar los mensajes de error.

    ```java
    private static void processError(ServiceBusErrorContext context, CountDownLatch countdownLatch) {
        System.out.printf("Error when receiving messages from namespace: '%s'. Entity: '%s'%n",
            context.getFullyQualifiedNamespace(), context.getEntityPath());

        if (!(context.getException() instanceof ServiceBusException)) {
            System.out.printf("Non-ServiceBusException occurred: %s%n", context.getException());
            return;
        }

        ServiceBusException exception = (ServiceBusException) context.getException();
        ServiceBusFailureReason reason = exception.getReason();

        if (reason == ServiceBusFailureReason.MESSAGING_ENTITY_DISABLED
            || reason == ServiceBusFailureReason.MESSAGING_ENTITY_NOT_FOUND
            || reason == ServiceBusFailureReason.UNAUTHORIZED) {
            System.out.printf("An unrecoverable error occurred. Stopping processing with reason %s: %s%n",
                reason, exception.getMessage());

            countdownLatch.countDown();
        } else if (reason == ServiceBusFailureReason.MESSAGE_LOCK_LOST) {
            System.out.printf("Message lock lost for message: %s%n", context.getException());
        } else if (reason == ServiceBusFailureReason.SERVICE_BUSY) {
            try {
                // Choosing an arbitrary amount of time to wait until trying again.
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                System.err.println("Unable to sleep for period of time");
            }
        } else {
            System.out.printf("Error source %s, reason %s, message: %s%n", context.getErrorSource(),
                reason, context.getException());
        }
    }  
    ```
2. Actualice el método `main` para invocar los métodos `sendMessage`, `sendMessageBatch` y `receiveMessages`, y para iniciar `InterruptedException`.     

    ```java
    public static void main(String[] args) throws InterruptedException {        
        sendMessage();
        sendMessageBatch();
        receiveMessages();
    }   
    ```

## <a name="run-the-app"></a>Ejecutar la aplicación
Al ejecutar la aplicación, verá los siguientes mensajes en la ventana de la consola. 

```console
Sent a single message to the queue: myqueue
Sent a batch of messages to the queue: myqueue
Starting the processor
Processing message. Session: 88d961dd801f449e9c3e0f8a5393a527, Sequence #: 1. Contents: Hello, World!
Processing message. Session: e90c8d9039ce403bbe1d0ec7038033a0, Sequence #: 2. Contents: First message
Processing message. Session: 311a216a560c47d184f9831984e6ac1d, Sequence #: 3. Contents: Second message
Processing message. Session: f9a871be07414baf9505f2c3d466c4ab, Sequence #: 4. Contents: Third message
Stopping and closing the processor
```

En la página **Información general** del espacio de nombres de Service Bus en Azure Portal, verá el recuento de mensajes **entrantes** y **salientes**. Es posible que tenga que esperar alrededor de un minuto y luego actualizar la página para ver los valores más recientes. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/overview-incoming-outgoing-messages.png" alt-text="Recuento de mensajes entrantes y salientes" lightbox="./media/service-bus-java-how-to-use-queues/overview-incoming-outgoing-messages.png":::

Seleccione la cola en esta página **Información general** para ir a la página **Cola de Service Bus**. En esta página también verá el recuento de mensajes **entrantes** y **salientes**. También verá otra información, como el **tamaño actual** de la cola, el **tamaño máximo** o el **recuento de mensajes activos**. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/queue-details.png" alt-text="Detalles de la cola" lightbox="./media/service-bus-java-how-to-use-queues/queue-details.png":::



## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación y los ejemplos siguientes:

- [Biblioteca cliente de Azure Service Bus para Java: léame](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/servicebus/azure-messaging-servicebus/README.md).
- [Ejemplos en GitHub](/samples/azure/azure-sdk-for-java/servicebus-samples/)
- [Referencia de API de Java](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-messaging-servicebus/7.0.0/index.html)

[Azure SDK for Java]: /azure/developer/java/sdk/get-started
[Azure Toolkit for Eclipse]: /azure/developer/java/toolkit-for-eclipse/installation
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
