---
title: Envíe mensajes a un tema de Azure Service Bus y reciba mensajes de las suscripciones a ese tema (JavaScript)
description: Aprenda a escribir un programa de JavaScript que use la versión más reciente del paquete @azure/service-bus para enviar mensajes a un tema de Service Bus y recibirlos desde una suscripción al tema.
author: spelluru
ms.author: spelluru
ms.date: 11/09/2020
ms.topic: quickstart
ms.devlang: nodejs
ms.custom:
- devx-track-js
- mode-api
ms.openlocfilehash: 13664e2a465ae891666e9c296514dc18a14ed191
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111950204"
---
# <a name="send-messages-to-an-azure-service-bus-topic-and-receive-messages-from-subscriptions-to-the-topic-javascript"></a>Envíe mensajes a un tema de Azure Service Bus y reciba mensajes de las suscripciones a ese tema (JavaScript)
En este tutorial aprenderá a usar el paquete [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus) de un programa de JavaScript para enviar mensajes a un tema de Service Bus y recibirlos de una suscripción de Service Bus a ese tema.

## <a name="prerequisites"></a>Requisitos previos
- Suscripción a Azure. Para completar este tutorial, deberá tener una cuenta de Azure. Puede activar sus [ventajas de suscriptor a MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Siga los pasos de [Inicio rápido: Uso de Azure Portal para crear un tema de Service Bus y suscripciones a ese tema](service-bus-quickstart-topics-subscriptions-portal.md). Anote la cadena de conexión, el nombre del tema y un nombre de suscripción. Solo usará una suscripción para este inicio rápido. 

> [!NOTE]
> - En este tutorial se trabaja con ejemplos que puede copiar y ejecutar con [Nodejs](https://nodejs.org/). Para instrucciones sobre cómo crear una aplicación Node.js, consulte [Creación e implementación de una aplicación Node.js en un sitio web de Azure](../app-service/quickstart-nodejs.md) o [Servicio en la nube Node.js (con Windows PowerShell)](../cloud-services/cloud-services-nodejs-develop-deploy-app.md).

### <a name="use-node-package-manager-npm-to-install-the-package"></a>Uso del Administrador de paquetes para Node (NPM) para instalar el paquete
Para instalar el paquete de npm para Service Bus, abra un símbolo del sistema que tenga `npm` en la ruta de acceso, cambie el directorio a la carpeta en la que quiere colocar los ejemplos y ejecute este comando.

```bash
npm install @azure/service-bus
```

## <a name="send-messages-to-a-topic"></a>Envío de mensajes a un tema
En el código de ejemplo siguiente se muestra cómo enviar un lote de mensajes a un tema de Service Bus. Vea los comentarios de código para obtener más detalles. 

1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com/)
2. Cree un archivo denominado `sendtotopic.js` y pegue en él el código siguiente. Este código enviará un mensaje al tema.

    ```javascript
    const { ServiceBusClient } = require("@azure/service-bus");
    
    const connectionString = "<SERVICE BUS NAMESPACE CONNECTION STRING>"
    const topicName = "<TOPIC NAME>";
    
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
     
        // createSender() can also be used to create a sender for a queue.
        const sender = sbClient.createSender(topicName);
     
        try {
            // Tries to send all messages in a single batch.
            // Will fail if the messages cannot fit in a batch.
            // await sender.sendMessages(messages);
     
            // create a batch object
            let batch = await sender.createMessageBatch(); 
            for (let i = 0; i < messages.length; i++) {
                // for each message in the arry         
    
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
    
            // Send the last created batch of messages to the topic
            await sender.sendMessages(batch);
     
            console.log(`Sent a batch of messages to the topic: ${topicName}`);
                    
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
3. Reemplace `<SERVICE BUS NAMESPACE CONNECTION STRING>` por la cadena de conexión del espacio de nombres de Service Bus.
1. Reemplace `<TOPIC NAME>` por el nombre del tema. 
1. Después, ejecute el comando en un símbolo del sistema para ejecutar este archivo.

    ```console
    node sendtotopic.js 
    ```
1. Debería ver la siguiente salida.

    ```console
    Sent a batch of messages to the topic: mytopic
    ```

## <a name="receive-messages-from-a-subscription"></a>Recepción de mensajes de una suscripción
1. Abra el editor que prefiera, como [Visual Studio Code](https://code.visualstudio.com/)
2. Cree un archivo denominado **receivefromsubscription.js** y pegue en él el código siguiente. Vea los comentarios de código para obtener más detalles. 

    ```javascript
    const { delay, ServiceBusClient, ServiceBusMessage } = require("@azure/service-bus");
    
    const connectionString = "<SERVICE BUS NAMESPACE CONNECTION STRING>"
    const topicName = "<TOPIC NAME>";
    const subscriptionName = "<SUBSCRIPTION NAME>";
    
     async function main() {
        // create a Service Bus client using the connection string to the Service Bus namespace
        const sbClient = new ServiceBusClient(connectionString);
     
        // createReceiver() can also be used to create a receiver for a queue.
        const receiver = sbClient.createReceiver(topicName, subscriptionName);
    
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
        await delay(5000);
    
        await receiver.close(); 
        await sbClient.close();
    }
    
    // call the main function
    main().catch((err) => {
        console.log("Error occurred: ", err);
        process.exit(1);
     });    
    ```
3. Reemplace `<SERVICE BUS NAMESPACE CONNECTION STRING>` por la cadena de conexión al espacio de nombres. 
1. Reemplace `<TOPIC NAME>` por el nombre del tema. 
1. Reemplace `<SUBSCRIPTION NAME>` por el nombre de la suscripción al tema. 
1. Después, ejecute el comando en un símbolo del sistema para ejecutar este archivo.

    ```console
    node receivefromsubscription.js
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

En Azure Portal, vaya al espacio de nombres de Service Bus y seleccione el tema en el panel inferior para ver la página **Tema de Service Bus** de su tema. En esta página debería ver tres mensajes entrantes y tres salientes en el gráfico **Mensajes**. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/topic-page-portal.png" alt-text="Mensajes entrantes y salientes":::

Si ejecuta solo la aplicación de envío la próxima vez, en la página **Tema de Service Bus**, verá seis mensajes entrantes (3 nuevos) pero tres mensajes salientes. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/updated-topic-page.png" alt-text="Página del tema actualizado":::

En esta página, si selecciona una suscripción, irá a la página **Service Bus Subscription** (Suscripción de Service Bus). En esta página puede ver el recuento de mensajes activos, el de mensajes con problemas de entrega y mucho más. En este ejemplo hay tres mensajes activos que todavía no ha recibido ningún receptor. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/active-message-count.png" alt-text="Recuento de mensajes activos":::

## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación y los ejemplos siguientes: 

- [Biblioteca cliente de Azure Service Bus para JavaScript](https://www.npmjs.com/package/@azure/service-bus)
- [Ejemplos de JavaScript](/samples/azure/azure-sdk-for-js/service-bus-javascript/)
- [Ejemplos de TypeScript](/samples/azure/azure-sdk-for-js/service-bus-typescript/)
- [Documentación de referencia de API](/javascript/api/overview/azure/service-bus)