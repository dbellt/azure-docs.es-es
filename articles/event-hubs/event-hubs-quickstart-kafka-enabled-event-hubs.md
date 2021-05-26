---
title: 'Inicio rápido: Streaming de datos con Azure Event Hubs mediante el protocolo de Kafka'
description: 'Inicio rápido: En este artículo se proporciona información acerca de cómo hacer streaming en Azure Event Hubs mediante el protocolo de Kafka y las API.'
ms.topic: quickstart
ms.date: 05/10/2021
ms.openlocfilehash: 601f9293c99ab68e10b39e33d036961900835db5
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110379070"
---
# <a name="quickstart-data-streaming-with-event-hubs-using-the-kafka-protocol"></a>Inicio rápido: Streaming de datos con Event Hubs mediante el protocolo de Kafka
En esta guía de inicio rápido se muestra cómo transmitir a instancias de Event Hubs sin cambiar los clientes del protocolo o ejecutar sus propios clústeres. Aprenderá a usar los productores y los consumidores para hablar con instancias de Event Hubs solo con un cambio de configuración en las aplicaciones. 

> [!NOTE]
> Este ejemplo está disponible en [GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/java).

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía de inicio rápido, asegúrese de cumplir los siguientes requisitos previos:

* Lea el artículo [Event Hubs para Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md).
* Suscripción a Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.
* [Kit de desarrollo de Java (JDK) 1.7+](/azure/developer/java/fundamentals/java-jdk-long-term-support).
* [Descargue](https://maven.apache.org/download.cgi) e [instale](https://maven.apache.org/install.html) un archivo binario de Maven.
* [Git](https://www.git-scm.com/)


## <a name="create-an-event-hubs-namespace"></a>Creación de un espacio de nombres de Event Hubs
Al crear un espacio de nombres de Event Hubs, se habilita automáticamente el punto de conexión de Kafka para el espacio de nombres. Puede transmitir eventos a Event Hubs desde las aplicaciones que usan el protocolo de Kafka. Siga las instrucciones paso a paso de [Creación de un centro de eventos mediante Azure Portal](event-hubs-create.md) para crear un espacio de nombres de Event Hubs. Si usa un clúster dedicado, consulte [Creación de un espacio de nombres y un centro de eventos en un clúster dedicado](event-hubs-dedicated-cluster-create-portal.md#create-a-namespace-and-event-hub-within-a-cluster).

> [!NOTE]
> Event Hubs para Kafka no se admite en el nivel **Básico**.

## <a name="send-and-receive-messages-with-kafka-in-event-hubs"></a>Envío y recepción de mensajes con Kafka en Event Hubs

1. Clone el [repositorio de Azure Event Hubs para Kafka](https://github.com/Azure/azure-event-hubs-for-kafka).

2. Vaya a `azure-event-hubs-for-kafka/quickstart/java/producer`.

3. Actualice los detalles de configuración para el productor en `src/main/resources/producer.config` de la siguiente manera:

    **TLS/SSL:**

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";
    ```
    
    > [!IMPORTANT]
    > Reemplace `{YOUR.EVENTHUBS.CONNECTION.STRING}` por la cadena de conexión para el espacio de nombres de Event Hubs. Para obtener instrucciones sobre cómo obtener la cadena de conexión, consulte [Obtención de una cadena de conexión de Event Hubs](event-hubs-get-connection-string.md). A continuación se muestra un ejemplo de configuración: `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`

    **OAuth:**

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=OAUTHBEARER
    sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required;
    sasl.login.callback.handler.class=CustomAuthenticateCallbackHandler;
    ```    

    [Aquí](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth/java/appsecret/producer/src/main/java) puede encontrar el código fuente de la clase de controlador de ejemplo CustomAuthenticateCallbackHandler en GitHub.
4. Ejecute el código del productor y transmita los eventos a Event Hubs:
   
    ```shell
    mvn clean package
    mvn exec:java -Dexec.mainClass="TestProducer"                                    
    ```
    
5. Vaya a `azure-event-hubs-for-kafka/quickstart/java/consumer`.

6. Actualice los detalles de configuración para el consumidor en `src/main/resources/consumer.config` de la siguiente manera:
   
    **TLS/SSL:**

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";
    ```

    > [!IMPORTANT]
    > Reemplace `{YOUR.EVENTHUBS.CONNECTION.STRING}` por la cadena de conexión para el espacio de nombres de Event Hubs. Para obtener instrucciones sobre cómo obtener la cadena de conexión, consulte [Obtención de una cadena de conexión de Event Hubs](event-hubs-get-connection-string.md). A continuación se muestra un ejemplo de configuración: `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`

    **OAuth:**

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=OAUTHBEARER
    sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required;
    sasl.login.callback.handler.class=CustomAuthenticateCallbackHandler;
    ``` 

    [Aquí](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth/java/appsecret/consumer/src/main/java) puede encontrar el código fuente de la clase de controlador de ejemplo CustomAuthenticateCallbackHandler en GitHub.

    Puede encontrar todos los ejemplos de OAuth de Event Hubs para Kafka [aquí](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth).
7. Ejecute el código de consumidor y procese eventos desde el centro de eventos mediante los clientes de Kafka:

    ```java
    mvn clean package
    mvn exec:java -Dexec.mainClass="TestConsumer"                                    
    ```

Si el clúster de Kafka de Event Hubs tiene eventos, ahora comenzará a recibirlos desde el consumidor.

## <a name="next-steps"></a>Pasos siguientes
En este artículo se muestra cómo hacer streaming a instancias de Event Hubs sin cambiar los clientes del protocolo o ejecutar sus propios clústeres. Para más información, consulte [Guía del desarrollador de Apache Kafka para Azure Event Hubs](apache-kafka-developer-guide.md).