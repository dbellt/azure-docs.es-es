---
title: Guía del desarrollador de Apache Kafka para Event Hubs
description: En este artículo se proporcionan vínculos a artículos que describen cómo integrar las aplicaciones de Kafka en Azure Event Hubs.
ms.date: 06/23/2020
ms.topic: article
ms.openlocfilehash: 84fa40d7cc4b09f31d78665f901b49adacf3267f
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107303090"
---
# <a name="apache-kafka-developer-guide-for-azure-event-hubs"></a>Guía del desarrollador de Apache Kafka para Azure Event Hubs
En este artículo se proporcionan vínculos a artículos que describen cómo integrar las aplicaciones de Apache Kafka en Azure Event Hubs. 

## <a name="overview"></a>Información general
Event Hubs proporciona un punto de conexión de Kafka que las aplicaciones basadas en Kafka existentes pueden usar como alternativa a la ejecución de su propio clúster de Kafka. Event Hubs funciona con muchas de sus aplicaciones de Kafka actuales. Para obtener más información, consulte [Event Hubs para Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md).

## <a name="quickstarts"></a>Guías de inicio rápido
Puede encontrar guías de inicio rápido en GitHub y en este conjunto de contenido que le ayudarán a avanzar rápidamente en Event Hubs para Kafka.

### <a name="quickstarts-in-github"></a>Guías de inicio rápido en GitHub
Consulte las siguientes guías de inicio rápido en el repositorio **azure-event-hubs-for-kafka**: 

| Lenguaje/marco del cliente | Descripción | 
| ------------------------- | ----------- | 
| [.NET](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/dotnet) | <p>En esta guía de inicio rápido se muestra cómo crear y conectarse a un punto de conexión de Kafka para Event Hubs con un productor y un consumidor de ejemplo escritos en C# mediante .NET Core 2.0.</p><p>Este ejemplo se basa en el [cliente de .NET para Apache Kafka de Confluent](https://github.com/confluentinc/confluent-kafka-dotnet), modificado para su uso con Event Hubs para Kafka.</p> | 
| [Java](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/java) | En esta guía de inicio rápido se muestra cómo crear y conectarse a un punto de conexión de Kafka para Event Hubs con un productor y un consumidor de ejemplo escritos en Java. |
| [Node.js](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/node) | <p>En esta guía de inicio rápido se muestra cómo crear y conectarse a un punto de conexión de Kafka para Event Hubs con un productor y un consumidor de ejemplo escritos en Node.</p><p>En este ejemplo se usa la biblioteca [node-rdkafka](https://github.com/Blizzard/node-rdkafka). </p>| 
| [Python](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/python) | <p>En esta guía de inicio rápido se muestra cómo crear y conectarse a un punto de conexión de Kafka para Event Hubs con un productor y un consumidor de ejemplo escritos en python.</p><p>Este ejemplo se basa en el [cliente de Python para Apache Kafka de Confluent](https://github.com/confluentinc/confluent-kafka-python), modificado para su uso con Event Hubs para Kafka.</p>|
| [Go](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/go) | <p>En esta guía de inicio rápido se muestra cómo crear y conectarse a un punto de conexión de Kafka para Event Hubs con un productor y un consumidor de ejemplo escritos en Go.</p><p>Este ejemplo se basa en el [cliente de Golang para Apache Kafka de Confluent](https://github.com/confluentinc/confluent-kafka-go), modificado para su uso con Event Hubs para Kafka.</p>| 
| [Sarama Kafka Go](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/go-sarama-client) | En esta guía de inicio rápido se muestra cómo crear y conectarse a un punto de conexión de Kafka para Event Hubs con un productor y un consumidor de ejemplo escritos en Go mediante la biblioteca del [cliente de Sarama Kafka](https://github.com/Shopify/sarama). |
| [Kafka](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/kafka-cli) | En esta guía de inicio rápido se muestra cómo crear y conectarse a un punto de conexión de Kafka para Event Hubs mediante la CLI incluida en la distribución de Apache Kafka.| 
| [Kafkacat](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/kafkacat) | kafkacat es un consumidor y productor de línea de comandos que no es JVM, basados en librdkafka, popular debido a su velocidad y su pequeña superficie. Esta guía de inicio rápido contiene una configuración de ejemplo y varios comandos sencillos de kafkacat de ejemplo. | 
 
### <a name="quickstarts-in-docs"></a>Guías de inicio rápido en DOCS
Consulte la guía de inicio rápido: [Streaming de datos con Event Hubs mediante el protocolo de Kafka](event-hubs-quickstart-kafka-enabled-event-hubs.md) en este conjunto de contenido, que proporciona instrucciones paso a paso sobre cómo transmitir en Event Hubs. Aprenderá a usar los productores y los consumidores para hablar con instancias de Event Hubs solo con un cambio de configuración en las aplicaciones. 


## <a name="tutorials"></a>Tutoriales 

### <a name="tutorials-in-github"></a>Tutoriales en GitHub
Consulte los siguientes tutoriales en GitHub:

| Tutorial | Descripción | 
| ------------------------- | ----------- | 
| [Akka](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/akka/java) | En este tutorial se muestra cómo conectar Akka Streams a instancias de Event Hubs habilitadas para Kafka sin cambiar los clientes del protocolo ni ejecutar sus propios clústeres. Hay dos tutoriales independientes que usan los lenguajes de programación **Java** y **Scala**. | 
| [Conexión](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/connect) | En este documento encontrará una guía para la integración de Kafka Connect en Azure Event Hubs y la implementación de conectores básicos FileStreamSource y FileStreamSink. Aunque estos conectores no están pensados para su uso en producción, muestran un escenario de un extremo a otro de Kafka Connect en el que Azure Event Hubs se enmascara como agente de Kafka.| 
| [Filebeat](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/filebeat) | En este documento encontrará una guía para la integración de Filebeat y Event Hubs a través de la salida Kafka de Filebeat. | 
| [Flink](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/flink) | En este tutorial se muestra cómo conectar Apache Flink a instancias de Event Hubs habilitadas para Kafka sin cambiar los clientes del protocolo ni ejecutar sus propios clústeres. | 
| [Fluentd](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/fluentd) | En este documento encontrará una guía para la integración de Fluentd y Event Hubs con el complemento de salida de `out_kafka` para Fluentd. |
| [Interop](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/interop) | En este tutorial se muestra cómo intercambiar eventos entre consumidores y productores mediante distintos protocolos. |
| [Logstash](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/logstash) | En este tutorial encontrará una guía para la integración de Logstash en Event Hubs habilitado para Kafka mediante complementos de entrada/salida de Logstash Kafka. | 
| [MirrorMaker](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/mirror-maker) | En este tutorial se muestra cómo un centro de eventos y Kafka MirrorMaker se pueden integrar en una canalización de Kafka existente en Azure al crear un reflejo del flujo de entrada de Kafka en el servicio Event Hubs. |
| [NiFi](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/nifi) | En este tutorial se muestra cómo conectar Apache NiFi a un espacio de nombres de Event Hubs. | 
| [OAuth](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth) | En las guías de inicio rápido se muestra cómo crear y conectarse a un punto de conexión de Kafka para Event Hubs con un productor y un consumidor de ejemplo escritos en los lenguajes de programación Go y Java. |
| [Confluent's Schema Registry](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/schema-registry) | En este tutorial encontrará una guía para la integración de Schema Registry y Event Hubs para Kafka. | 
| [Spark](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/spark) | En este tutorial se muestra cómo conectar su aplicación Spark a un centro de eventos sin cambiar los clientes del protocolo ni ejecutar sus propios clústeres de Kafka. | 

### <a name="tutorials-in-docs"></a>Tutoriales en DOCS
Además, consulte el tutorial: [Procesamiento de eventos de Apache Kafka para Event Hubs mediante Stream Analytics](event-hubs-kafka-stream-analytics.md) en este conjunto de contenido, que muestra cómo transmitir datos a Event Hubs y procesarlos con Azure Stream Analytics.

## <a name="how-to-guides"></a>Guías paso a paso
Consulte las siguientes guías paso a paso en nuestra documentación:

| Artículo | Descripción | 
| ------- | ----------- | 
| [Reflejo de un agente de Kafka en un centro de eventos](event-hubs-kafka-mirror-maker-tutorial.md) | Muestra cómo reflejar un agente de Kafka en un centro de eventos mediante Kafka MirrorMaker. |
| [Conexión de Apache Spark a un centro de eventos](event-hubs-kafka-spark-tutorial.md) | Le guía a través de la conexión de la aplicación de Spark a Event Hubs para el streaming en tiempo real. |
| [Conexión de Apache Flink a un centro de eventos](event-hubs-kafka-flink-tutorial.md) | Se muestra cómo conectar Apache Flink a un centro de eventos sin cambiar los clientes del protocolo o ejecutar sus propios clústeres. |
| [Integración de Apache Kafka Connect en Event Hubs (versión preliminar)](event-hubs-kafka-connect-tutorial.md) | Le guía por la integración de Kafka Connect con un centro de eventos y la implementación de conectores básicos FileStreamSource y FileStreamSink. |
| [Conexión de Akka Streams a un centro de eventos](event-hubs-kafka-akka-streams-tutorial.md) | Muestra cómo conectar Akka Streams a un centro de eventos sin cambiar los clientes del protocolo ni ejecutar sus propios clústeres. |
| [Cómo usar el iniciador de Spring Boot para Apache Kafka con Azure Event Hubs](/azure/developer/java/spring-framework/configure-spring-cloud-stream-binder-java-app-kafka-azure-event-hub) | Muestra cómo configurar una aplicación de Spring Cloud Stream Binder basada en Java creada con Spring Boot Initializer para usar Apache Kafka con Azure Event Hubs. |

## <a name="next-steps"></a>Pasos siguientes
Revise los ejemplos del repositorio de GitHub [azure-event-hubs-for-kafka](https://github.com/Azure/azure-event-hubs-for-kafka) en las carpetas de guías de inicio rápido y tutoriales.

Consulte también los siguientes artículos:

- [Guía de solución de problemas de Apache Kafka para Event Hubs](apache-kafka-troubleshooting-guide.md)
- [Preguntas más frecuentes: Event Hubs para Apache Kafka](apache-kafka-frequently-asked-questions.yml)
- [Guía de migración de Kafka para Event Hubs](apache-kafka-migration-guide.md)



