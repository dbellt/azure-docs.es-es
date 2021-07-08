---
title: Uso de colas de azure/service-bus en JavaScript
description: Aprenda a escribir un programa de JavaScript que use la versión más reciente del paquete @azure/service-bus para enviar mensajes a una cola de Service Bus, y recibirlos desde ella.
author: spelluru
ms.author: spelluru
ms.date: 11/09/2020
ms.topic: quickstart
ms.devlang: nodejs
ms.custom:
- devx-track-js
- mode-api
ms.openlocfilehash: fb84dd12dd8e7a5b60d77e7a4895ae5b5d67cde6
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110652279"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-javascript"></a>Envío de mensajes a las colas de Azure Service Bus (JavaScript) y recepción desde allí
En este tutorial aprenderá a usar el paquete [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus) de un programa de JavaScript para enviar mensajes a una cola de Service Bus, y recibirlos de ella.

## <a name="prerequisites"></a>Requisitos previos
- Suscripción a Azure. Para completar este tutorial, deberá tener una cuenta de Azure. Puede activar sus [ventajas de suscriptor a MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Si no tiene una cola con la que trabajar, siga los pasos del artículo [Uso de Azure Portal para crear una cola de Service Bus](service-bus-quickstart-portal.md) para crear una. Anote la **cadena de conexión** para el espacio de nombres de Service Bus y el nombre de la **cola** que creó.

> [!NOTE]
> - En este tutorial se trabaja con ejemplos que puede copiar y ejecutar con [Nodejs](https://nodejs.org/). Para obtener instrucciones acerca de cómo crear una aplicación de Node.js, consulte [Creación e implementación de una aplicación Node.js en un sitio web de Azure](../app-service/quickstart-nodejs.md) o [Servicio en la nube Node.js (con Windows PowerShell)](../cloud-services/cloud-services-nodejs-develop-deploy-app.md).

### <a name="use-node-package-manager-npm-to-install-the-package"></a>Uso del Administrador de paquetes para Node (NPM) para instalar el paquete
Para instalar el paquete de npm para Service Bus, abra un símbolo del sistema que tenga `npm` en la ruta de acceso, cambie el directorio a la carpeta en la que quiere colocar los ejemplos y ejecute este comando.

```bash
npm install @azure/service-bus
```

## <a name="send-messages-to-a-queue"></a>mensajes a una cola
El código de ejemplo siguiente muestra cómo enviar un mensaje a una cola.

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com/).
2. Cree un archivo denominado `send.js` y pegue en él el código siguiente. Este código envía los nombres de los científicos como mensajes a la cola.

    ```javascript
    const { ServiceBusClient } = require("@azure/service-bus");
    
    // connection string to your Service Bus namespace
    const connectionString = "<CONNECTION STRING TO SERVICE BUS NAMESPACE>"

    // name of the queue
    const queueName = "<QUEUE NAME>"
    
    const messages = [
        { body: "Albert Einstein" },
        { body: "Werner Heisenberg" },
        { body: "Marie Curie" },
        { body: "Steven Hawking" },
        { body: "Isaac Newton" },
        { body: "Niels Bohr" },
        { body: "Michael Faraday" },
        { body: "Galileo Galilei" },
        { body: "Johannes Kepler" },
        { body: "Nikolaus Kopernikus" }
     ];
    
    async function main() {
        // create a Service Bus client using the connection string to the Service Bus namespace
        const sbClient = new ServiceBusClient(connectionString);
     
        // createSender() can also be used to create a sender for a topic.
        const sender = sbClient.createSender(queueName);
     
        try {
            // Tries to send all messages in a single batch.
            // Will fail if the messages cannot fit in a batch.
            // await sender.sendMessages(messages);
     
            // create a batch object
            let batch = await sender.createMessageBatch(); 
            for (let i = 0; i < messages.length; i++) {
                // for each message in the array            
    
                // try to add the message to the batch
                if (!batch.tryAddMessage(messages[i])) {            
                    // if it fails to add the message to the current batch
                    // send the current batch as it is full
                    await sender.sendMessages(batch);
    
                    // then, create a new batch 
                    batch = await sender.createMessageBatch();
     
                    // now, add the message failed to be added to the previous batch to this batch
                    if (!batch.tryAddMessage(messages[i])) {
                        // if it still can't be added to the batch, the message is probably too big to fit in a batch
                        throw new Error("Message too big to fit in a batch");
                    }
                }
            }
    
            // Send the last created batch of messages to the queue
            await sender.sendMessages(batch);
     
            console.log(`Sent a batch of messages to the queue: ${queueName}`);
                    
            // Close the sender
            await sender.close();
        } finally {
            await sbClient.close();
        }
    }
    
    // call the main function
    main().catch((err) => {
        console.log("Error occurred: ", err);
        process.exit(1);
     });
    ```
3. Reemplace `<CONNECTION STRING TO SERVICE BUS NAMESPACE>` por la cadena de conexión del espacio de nombres de Service Bus.
1. Reemplace `<QUEUE NAME>` por el nombre de la cola. 
1. Después, ejecute el comando en un símbolo del sistema para ejecutar este archivo.

    ```console
    node send.js 
    ```
1. Debería ver la siguiente salida.

    ```console
    Sent a batch of messages to the queue: myqueue
    ```

## <a name="receive-messages-from-a-queue"></a>mensajes de una cola

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com/)
2. Cree un archivo denominado `receive.js` y pegue en él el código siguiente.

    ```javascript
    const { delay, ServiceBusClient, ServiceBusMessage } = require("@azure/service-bus");

    // connection string to your Service Bus namespace
    const connectionString = "<CONNECTION STRING TO SERVICE BUS NAMESPACE>"

    // name of the queue
    const queueName = "<QUEUE NAME>"

     async function main() {
        // create a Service Bus client using the connection string to the Service Bus namespace
        const sbClient = new ServiceBusClient(connectionString);
     
        // createReceiver() can also be used to create a receiver for a subscription.
        const receiver = sbClient.createReceiver(queueName);
    
        // function to handle messages
        const myMessageHandler = async (messageReceived) => {
            console.log(`Received message: ${messageReceived.body}`);
        };
    
        // function to handle any errors
        const myErrorHandler = async (error) => {
            console.log(error);
        };
    
        // subscribe and specify the message and error handlers
        receiver.subscribe({
            processMessage: myMessageHandler,
            processError: myErrorHandler
        });
    
        // Waiting long enough before closing the sender to send messages
        await delay(20000);
    
        await receiver.close(); 
        await sbClient.close();
    }    
    // call the main function
    main().catch((err) => {
        console.log("Error occurred: ", err);
        process.exit(1);
     });
    ```
3. Reemplace `<CONNECTION STRING TO SERVICE BUS NAMESPACE>` por la cadena de conexión del espacio de nombres de Service Bus.
1. Reemplace `<QUEUE NAME>` por el nombre de la cola. 
1. Después, ejecute el comando en un símbolo del sistema para ejecutar este archivo.

    ```console
    node receive.js
    ```
1. Debería ver la siguiente salida.

    ```console
    Received message: Albert Einstein
    Received message: Werner Heisenberg
    Received message: Marie Curie
    Received message: Steven Hawking
    Received message: Isaac Newton
    Received message: Niels Bohr
    Received message: Michael Faraday
    Received message: Galileo Galilei
    Received message: Johannes Kepler
    Received message: Nikolaus Kopernikus
    ```

En la página **Información general** del espacio de nombres de Service Bus en Azure Portal, verá el recuento de mensajes **entrantes** y **salientes**. Es posible que tenga que esperar alrededor de un minuto y luego actualizar la página para ver los valores más recientes. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/overview-incoming-outgoing-messages.png" alt-text="Recuento de mensajes entrantes y salientes":::

Seleccione la cola en esta página **Información general** para ir a la página **Cola de Service Bus**. En esta página también verá el recuento de mensajes **entrantes** y **salientes**. También verá otra información, como el **tamaño actual** de la cola, el **tamaño máximo** o el **recuento de mensajes activos**. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/queue-details.png" alt-text="Detalles de la cola":::
## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación y los ejemplos siguientes: 

- [Biblioteca cliente de Azure Service Bus para JavaScript](https://www.npmjs.com/package/@azure/service-bus)
- [Ejemplos de JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/servicebus/service-bus/samples/v7/javascript)
- [Ejemplos de TypeScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/servicebus/service-bus/samples/v7/typescript)
- [Documentación de referencia de API](/javascript/api/overview/azure/service-bus)
