---
title: Uso de temas y suscripciones de Service Bus con el paquete azure-servicebus de Python, versión 7.0.0
description: En este artículo se muestra cómo usar Python para enviar mensajes a un tema y recibirlos de la suscripción.
documentationcenter: python
author: spelluru
ms.author: spelluru
ms.date: 11/18/2020
ms.topic: quickstart
ms.devlang: python
ms.custom:
- devx-track-python
- mode-api
ms.openlocfilehash: 49e80e277c6df5372341293861d5bda0580f3e8c
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107537170"
---
# <a name="send-messages-to-an-azure-service-bus-topic-and-receive-messages-from-subscriptions-to-the-topic-python"></a>Envío de mensajes a un tema de Azure Service Bus y recepción de mensajes de las suscripciones a ese tema (Python)
En este artículo se muestra cómo usar Python para enviar mensajes a un tema de Service Bus y recibirlos de una suscripción a ese tema. 

## <a name="prerequisites"></a>Requisitos previos
- Suscripción a Azure. Puede activar sus [beneficios de suscriptor de Visual Studio o MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Siga los pasos de [Inicio rápido: Uso de Azure Portal para crear un tema de Service Bus y suscripciones a ese tema](service-bus-quickstart-topics-subscriptions-portal.md). Anote la cadena de conexión, el nombre del tema y un nombre de suscripción. Solo usará una suscripción para este inicio rápido. 
- Python 2.7 o una versión posterior, con el paquete [SDK de Azure Python] [paquete de Azure Python] instalado. Para obtener más información, consulte la [guía de instalación de Python](/azure/developer/python/azure-sdk-install).

## <a name="send-messages-to-a-topic"></a>Envío de mensajes a un tema

1. Agregue la siguiente instrucción import. 

    ```python
    from azure.servicebus import ServiceBusClient, ServiceBusMessage
    ```
2. Agregue las constantes siguientes. 

    ```python
    CONNECTION_STR = "<NAMESPACE CONNECTION STRING>"
    TOPIC_NAME = "<TOPIC NAME>"
    SUBSCRIPTION_NAME = "<SUBSCRIPTION NAME>"
    ```
    
    > [!IMPORTANT]
    > - Reemplace `<NAMESPACE CONNECTION STRING>` por la cadena de conexión para el espacio de nombres.
    > - Reemplace `<TOPIC NAME>` por el nombre del tema.
    > - Reemplace `<SUBSCRIPTION NAME>` por el nombre de la suscripción al tema. 
3. Agregue un método para enviar un solo mensaje.

    ```python
    def send_single_message(sender):
        # create a Service Bus message
        message = ServiceBusMessage("Single Message")
        # send the message to the topic
        sender.send_messages(message)
        print("Sent a single message")
    ```

    El remitente es un objeto que actúa como cliente para el tema que ha creado. Lo creará más adelante y lo enviará como argumento a esta función. 
4. Agregue un método para enviar una lista de mensajes.

    ```python
    def send_a_list_of_messages(sender):
        # create a list of messages
        messages = [ServiceBusMessage("Message in list") for _ in range(5)]
        # send the list of messages to the topic
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
        # send the batch of messages to the topic
        sender.send_messages(batch_message)
        print("Sent a batch of 10 messages")
    ```
6. Cree un cliente de Service Bus y un objeto de remitente de tema para enviar mensajes.

    ```python
    # create a Service Bus client using the connection string
    servicebus_client = ServiceBusClient.from_connection_string(conn_str=CONNECTION_STR, logging_enable=True)
    with servicebus_client:
        # get a Topic Sender object to send messages to the topic
        sender = servicebus_client.get_topic_sender(topic_name=TOPIC_NAME)
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
 
## <a name="receive-messages-from-a-subscription"></a>Recepción de mensajes de una suscripción
Agregue el siguiente código después de la instrucción de impresión. Este código recibe continuamente mensajes nuevos, hasta que no recibe mensajes nuevos durante 5 (`max_wait_time`) segundos. 

```python
with servicebus_client:
    # get the Subscription Receiver object for the subscription    
    receiver = servicebus_client.get_subscription_receiver(topic_name=TOPIC_NAME, subscription_name=SUBSCRIPTION_NAME, max_wait_time=5)
    with receiver:
        for msg in receiver:
            print("Received: " + str(msg))
            # complete the message so that the message is removed from the subscription
            receiver.complete_message(msg)
```

## <a name="full-code"></a>Código completo

```python
from azure.servicebus import ServiceBusClient, ServiceBusMessage

CONNECTION_STR = "<NAMESPACE CONNECTION STRING>"
TOPIC_NAME = "<TOPIC NAME>"
SUBSCRIPTION_NAME = "<SUBSCRIPTION NAME>"

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
    sender = servicebus_client.get_topic_sender(topic_name=TOPIC_NAME)
    with sender:
        send_single_message(sender)
        send_a_list_of_messages(sender)
        send_batch_message(sender)

print("Done sending messages")
print("-----------------------")

with servicebus_client:
    receiver = servicebus_client.get_subscription_receiver(topic_name=TOPIC_NAME, subscription_name=SUBSCRIPTION_NAME, max_wait_time=5)
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

Seleccione el tema en el panel inferior para ver la página **Tema de Service Bus** de su tema. En esta página debería ver tres mensajes entrantes y tres salientes en el gráfico **Mensajes**. 

:::image type="content" source="./media/service-bus-python-how-to-use-topics-subscriptions/topic-page-portal.png" alt-text="Mensajes entrantes y salientes":::

En esta página, si selecciona una suscripción, irá a la página **Service Bus Subscription** (Suscripción de Service Bus). En esta página puede ver el recuento de mensajes activos, el de mensajes con problemas de entrega y mucho más. En este ejemplo se han recibido todos los mensajes, por lo que el número de mensajes activos es cero. 

:::image type="content" source="./media/service-bus-python-how-to-use-topics-subscriptions/active-message-count.png" alt-text="Recuento de mensajes activos":::

Si marca el código de recepción, verá que el número de mensajes activos es 16. 

:::image type="content" source="./media/service-bus-python-how-to-use-topics-subscriptions/active-message-count-2.png" alt-text="Recuento de mensajes activos: sin recepción":::

## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación y los ejemplos siguientes: 

- [Biblioteca cliente de Azure Service Bus para Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/servicebus/azure-servicebus).
- [Ejemplos](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/servicebus/azure-servicebus/samples). 
    - La carpeta **sync_samples** tiene ejemplos que muestran cómo interactuar con Service Bus de forma sincrónica. En este inicio rápido, ha usado este método. 
    - La carpeta **async_samples** tiene ejemplos que muestran cómo interactuar con Service Bus de forma asincrónica. 
- [Documentación de referencia de Azure Service Bus](/python/api/azure-servicebus/azure.servicebus?preserve-view=true)
