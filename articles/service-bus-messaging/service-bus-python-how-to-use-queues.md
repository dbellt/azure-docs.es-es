---
title: Uso de colas de Service Bus con el paquete azure-servicebus de Python, versión 7.0.0
description: En este artículo se muestra cómo usar Python para enviar mensajes de las colas de Azure Service Bus y recibirlos.
documentationcenter: python
author: spelluru
ms.author: spelluru
ms.date: 11/18/2020
ms.topic: quickstart
ms.devlang: python
ms.custom:
- seo-python-october2019
- devx-track-python
- mode-api
ms.openlocfilehash: e982b3f36dbfe2144170d53d07473201abb014ef
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107534972"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-python"></a>Envío de mensajes desde colas de Azure Service Bus (Python) y recepción
En este artículo se muestra cómo usar Python para enviar mensajes de las colas de Azure Service Bus y recibirlos. 

## <a name="prerequisites"></a>Requisitos previos
- Suscripción a Azure. Puede activar sus [beneficios de suscriptor de Visual Studio o MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Si no tiene una cola con la que trabajar, siga los pasos del artículo [Uso de Azure Portal para crear una cola de Service Bus](service-bus-quickstart-portal.md) para crear una. Anote la **cadena de conexión** para el espacio de nombres de Service Bus y el nombre de la **cola** que creó.
- Python 2.7 o una versión posterior, con el paquete [Python Azure Service Bus](https://pypi.python.org/pypi/azure-servicebus) instalado. Para obtener más información, consulte la [guía de instalación de Python](/azure/developer/python/azure-sdk-install). 

## <a name="send-messages-to-a-queue"></a>mensajes a una cola

1. Agregue la siguiente instrucción import. 

    ```python
    from azure.servicebus import ServiceBusClient, ServiceBusMessage
    ```
2. Agregue las constantes siguientes. 

    ```python
    CONNECTION_STR = "<NAMESPACE CONNECTION STRING>"
    QUEUE_NAME = "<QUEUE NAME>"
    ```

    > [!IMPORTANT]
    > - Reemplace `<NAMESPACE CONNECTION STRING>` por la cadena de conexión del espacio de nombres de Service Bus.
    > - Reemplace `<QUEUE NAME>` por el nombre de la cola. 
3. Agregue un método para enviar un solo mensaje.

    ```python
    def send_single_message(sender):
        # create a Service Bus message
        message = ServiceBusMessage("Single Message")
        # send the message to the queue
        sender.send_messages(message)
        print("Sent a single message")
    ```

    El remitente es un objeto que actúa como cliente para la cola que ha creado. Lo creará más adelante y lo enviará como argumento a esta función. 
4. Agregue un método para enviar una lista de mensajes.

    ```python
    def send_a_list_of_messages(sender):
        # create a list of messages
        messages = [ServiceBusMessage("Message in list") for _ in range(5)]
        # send the list of messages to the queue
        sender.send_messages(messages)
        print("Sent a list of 5 messages")
    ```
5. Agregue un método para enviar un lote de mensajes.

    ```python
    def send_batch_message(sender):
        # create a batch of messages
        batch_message = sender.create_message_batch()
        for _ in range(10):
            try:
                # add a message to the batch
                batch_message.add_message(ServiceBusMessage("Message inside a ServiceBusMessageBatch"))
            except ValueError:
                # ServiceBusMessageBatch object reaches max_size.
                # New ServiceBusMessageBatch object can be created here to send more data.
                break
        # send the batch of messages to the queue
        sender.send_messages(batch_message)
        print("Sent a batch of 10 messages")
    ```
6. Cree un cliente de Service Bus y un objeto de remitente de cola para enviar mensajes.

    ```python
    # create a Service Bus client using the connection string
    servicebus_client = ServiceBusClient.from_connection_string(conn_str=CONNECTION_STR, logging_enable=True)
    with servicebus_client:
        # get a Queue Sender object to send messages to the queue
        sender = servicebus_client.get_queue_sender(queue_name=QUEUE_NAME)
        with sender:
            # send one message        
            send_single_message(sender)
            # send a list of messages
            send_a_list_of_messages(sender)
            # send a batch of messages
            send_batch_message(sender)
    
    print("Done sending messages")
    print("-----------------------")
    ```
 
## <a name="receive-messages-from-a-queue"></a>mensajes de una cola
Agregue el siguiente código después de la instrucción de impresión. Este código recibe continuamente mensajes nuevos, hasta que no recibe mensajes nuevos durante 5 (`max_wait_time`) segundos. 

```python
with servicebus_client:
    # get the Queue Receiver object for the queue
    receiver = servicebus_client.get_queue_receiver(queue_name=QUEUE_NAME, max_wait_time=5)
    with receiver:
        for msg in receiver:
            print("Received: " + str(msg))
            # complete the message so that the message is removed from the queue
            receiver.complete_message(msg)
```

## <a name="full-code"></a>Código completo

```python
# import os
from azure.servicebus import ServiceBusClient, ServiceBusMessage

CONNECTION_STR = "<NAMESPACE CONNECTION STRING>"
QUEUE_NAME = "<QUEUE NAME>"

def send_single_message(sender):
    message = ServiceBusMessage("Single Message")
    sender.send_messages(message)
    print("Sent a single message")

def send_a_list_of_messages(sender):
    messages = [ServiceBusMessage("Message in list") for _ in range(5)]
    sender.send_messages(messages)
    print("Sent a list of 5 messages")

def send_batch_message(sender):
    batch_message = sender.create_message_batch()
    for _ in range(10):
        try:
            batch_message.add_message(ServiceBusMessage("Message inside a ServiceBusMessageBatch"))
        except ValueError:
            # ServiceBusMessageBatch object reaches max_size.
            # New ServiceBusMessageBatch object can be created here to send more data.
            break
    sender.send_messages(batch_message)
    print("Sent a batch of 10 messages")

servicebus_client = ServiceBusClient.from_connection_string(conn_str=CONNECTION_STR, logging_enable=True)

with servicebus_client:
    sender = servicebus_client.get_queue_sender(queue_name=QUEUE_NAME)
    with sender:
        send_single_message(sender)
        send_a_list_of_messages(sender)
        send_batch_message(sender)

print("Done sending messages")
print("-----------------------")

with servicebus_client:
    receiver = servicebus_client.get_queue_receiver(queue_name=QUEUE_NAME, max_wait_time=5)
    with receiver:
        for msg in receiver:
            print("Received: " + str(msg))
            receiver.complete_message(msg)
```

## <a name="run-the-app"></a>Ejecutar la aplicación
Al ejecutar la aplicación, debería ver la salida siguiente: 

```console
Sent a single message
Sent a list of 5 messages
Sent a batch of 10 messages
Done sending messages
-----------------------
Received: Single Message
Received: Message in list
Received: Message in list
Received: Message in list
Received: Message in list
Received: Message in list
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
```

En Azure Portal, vaya al espacio de nombres de Service Bus. En la página **Información general**, compruebe que el recuento de mensajes **entrantes** y **salientes** es 16. Si no los ve, actualice la página al cabo de unos minutos. 

:::image type="content" source="./media/service-bus-python-how-to-use-queues/overview-incoming-outgoing-messages.png" alt-text="Recuento de mensajes entrantes y salientes":::

Seleccione la cola en esta página **Información general** para ir a la página **Cola de Service Bus**. En esta página también verá el recuento de mensajes **entrantes** y **salientes**. También verá otra información, como el **tamaño actual** de la cola y el **recuento de mensajes activos**. 

:::image type="content" source="./media/service-bus-python-how-to-use-queues/queue-details.png" alt-text="Detalles de la cola":::


## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación y los ejemplos siguientes: 

- [Biblioteca cliente de Azure Service Bus para Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/servicebus/azure-servicebus).
- [Ejemplos](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/servicebus/azure-servicebus/samples). 
    - La carpeta **sync_samples** tiene ejemplos que muestran cómo interactuar con Service Bus de forma sincrónica. En este inicio rápido, ha usado este método. 
    - La carpeta **async_samples** tiene ejemplos que muestran cómo interactuar con Service Bus de forma asincrónica. 
- [Documentación de referencia de Azure Service Bus](/python/api/azure-servicebus/azure.servicebus?preserve-view=true)
