---
title: 'Uso del centro de eventos desde una aplicación de Apache Kafka: Azure Event Hubs | Microsoft Docs'
description: En este artículo se proporciona información sobre la compatibilidad de Apache Kafka con Azure Event Hubs.
ms.topic: article
ms.date: 09/25/2020
ms.openlocfilehash: 2e6e1defffd012e524044bc427788fc8a1bcc53a
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110376090"
---
# <a name="use-azure-event-hubs-from-apache-kafka-applications"></a>Uso de Azure Event Hubs desde aplicaciones de Apache Kafka
Event Hubs proporciona un punto de conexión compatible con las API de productor y consumidor de Apache Kafka® que pueden usar la mayoría de las aplicaciones cliente de Apache Kafka existentes como alternativa a la ejecución de su propio clúster de Apache Kafka. Event Hubs admite clientes de API de productor y consumidor de Apache Kafka en la versión 1.0 y posteriores.

> [!VIDEO https://www.youtube.com/embed/UE1WgB96_fc]

## <a name="what-does-event-hubs-for-kafka-provide"></a>¿Qué proporciona Event Hubs para Kafka?

La característica Event Hubs para Apache Kafka proporciona un protocolo principal sobre Azure Event Hubs que es compatible con los clientes de Apache Kafka creados para Apache Kafka 1.0 y versiones posteriores, y admite tanto la lectura como la escritura en Event Hubs, lo que es equivalente a los temas de Apache Kafka. 

A menudo, puede utilizar el punto de conexión de Kafka para Event Hubs desde sus aplicaciones sin cambios en el código en comparación con la configuración de Kafka existente y modificar solo la configuración: Actualice la cadena de conexión en las configuraciones para que apunte al punto de conexión de Kafka que se expone en el evento, en lugar de apuntar al clúster de Kafka. A partir de ahí, puede empezar a hacer streaming de los eventos desde las aplicaciones que usen el protocolo Kafka en Event Hubs. 

Conceptualmente, Kafka y Event Hubs son muy similares: son registros con particiones creados para la transmisión de datos, de modo que el cliente controla qué parte del registro retenido desea leer. En la tabla siguiente, se asignan conceptos entre Kafka y Event Hubs.

### <a name="kafka-and-event-hub-conceptual-mapping"></a>Asignación conceptual de Kafka y Event Hubs

| Concepto de Kafka | Concepto de Event Hubs|
| --- | --- |
| Clúster | Espacio de nombres |
| Tema | Centro de eventos |
| Partition | Partition|
| Grupo de consumidores | Grupo de consumidores |
| Offset | Offset|

### <a name="key-differences-between-apache-kafka-and-event-hubs"></a>Diferencias principales entre Apache Kafka y Event Hubs

[Apache Kafka](https://kafka.apache.org/) es el software que normalmente necesita para instalar y trabajar, mientras que Event Hubs es un servicio nativo de la nube totalmente administrado. No hay ningún servidor, discos o red que administrar o supervisar ni ningún agente que configurar o tener en cuenta. Puede crear un espacio de nombres, que es un punto de conexión con un nombre de dominio completo y, a continuación, crear instancias de Event Hubs (temas) dentro de ese espacio de nombres. 

Para más información acerca de Event Hubs y de los espacios de nombres, consulte las [características de Event Hubs](event-hubs-features.md#namespace). Como servicio en la nube, Event Hubs usa una única dirección IP virtual estable como punto de conexión, de forma que los clientes no necesitan conocer los agentes o equipos de un clúster. Aunque Event Hubs implementa el mismo protocolo, esta diferencia significa que todo el tráfico de Kafka para todas las particiones se enruta de forma predecible a través de este punto de conexión, en lugar de requerir acceso de firewall para todos los agentes de un clúster.   

La escala en Event Hubs se controla por el número de [unidades de rendimiento](event-hubs-scalability.md#throughput-units) o [unidades de procesamiento](event-hubs-scalability.md#processing-units) que se adquieren; cada unidad de rendimiento da derecho a 1 MB por segundo, o 1000 eventos por segundo, de entrada y dos veces ese volumen de salida. Event Hubs puede escalar verticalmente de forma automática las unidades de rendimiento o procesamiento cuando se alcanza el límite si se usa la característica de [inflado automático](event-hubs-auto-inflate.md), que también funciona con la compatibilidad con protocolos de Apache Kafka.  

### <a name="is-apache-kafka-the-right-solution-for-your-workload"></a>¿Es Apache Kafka la solución adecuada para su carga de trabajo?

Tras la creación de aplicaciones con Apache Kafka, resultará útil comprender que Azure Event Hubs forma parte de una flota de servicios que también incluye [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)y [Azure Event Grid](../event-grid/overview.md). 

Aunque algunos proveedores de distribuciones comerciales de Apache Kafka pueden sugerir que Apache Kafka es un servicio integral para todas las necesidades de plataforma de mensajería, la realidad es que Apache Kafka no implementa, por ejemplo, el patrón de cola de [consumidor en competencia](/azure/architecture/patterns/competing-consumers), no ofrece compatibilidad con el patrón de [publicación y suscripción](/azure/architecture/patterns/publisher-subscriber) a un nivel que permita a los suscriptores acceder a los mensajes entrantes en función de reglas evaluadas por el servidor que no sean simples desplazamientos, y no tiene instalaciones para realizar el seguimiento del ciclo de vida de un trabajo iniciado por un mensaje u omitir mensajes erróneos en una cola de mensajes fallidos; características que resultan básicas en muchos escenarios de mensajería empresarial.

Para comprender las diferencias entre los patrones y saber qué servicio es más adecuado para cada patrón, revise la guía [Opciones de mensajería asincrónica en Azure](/azure/architecture/guide/technology-choices/messaging). Como usuario de Apache Kafka, es posible que descubra que las rutas de comunicación que ha realizado hasta la fecha con Kafka se pueden llevar a cabo con mucha menos complejidad básica y con funcionalidades más eficaces mediante Event Grid o Service Bus. 

Si necesita características específicas de Apache Kafka que no están disponibles mediante la interfaz de Event Hubs para Apache Kafka o si su modelo de implementación supera las [cuotas de Event Hubs](event-hubs-quotas.md), también puede ejecutar un [clúster de Apache Kafka nativo en Azure HDInsight](../hdinsight/kafka/apache-kafka-introduction.md).  

## <a name="security-and-authentication"></a>Seguridad y autenticación
Cada vez que se publican o consumen eventos de Event Hubs para Kafka, el cliente intenta acceder a los recursos de Event Hubs. Quiere asegurarse de que se accede a los recursos mediante una entidad autorizada. Al usar el protocolo Apache Kafka con los clientes, puede establecer la configuración para la autenticación y el cifrado mediante los mecanismos de SASL. El empleo de Event Hubs para Kafka exige cifrado TLS (ya que todos los datos en tránsito con Event Hubs están cifrados con TLS). Para ello, especifique la opción SASL_SSL en el archivo de configuración. 

Azure Event Hubs proporciona varias opciones para autorizar el acceso a los recursos protegidos. 

- OAuth 2.0
- Firma de acceso compartido (SAS)

#### <a name="oauth-20"></a>OAuth 2.0
Event Hubs se integra con Azure Active Directory (Azure AD), lo que proporciona un servidor de autorización centralizado compatible con **OAuth 2.0**. Con Azure AD, puede usar el control de acceso basado en rol de Azure (RBAC de Azure) para aplicar personalización avanzada de permisos a las identidades de cliente. Puede usar esta característica con los clientes de Kafka si especifica **SASL_SSL** para el protocolo y **OAUTHBEARER** para el mecanismo. Para obtener detalles sobre los roles y los niveles de Azure para el control de acceso, consulte [Autorización del acceso con Azure AD](authorize-access-azure-active-directory.md).

```properties
bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
security.protocol=SASL_SSL
sasl.mechanism=OAUTHBEARER
sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required;
sasl.login.callback.handler.class=CustomAuthenticateCallbackHandler
```

#### <a name="shared-access-signature-sas"></a>Firma de acceso compartido (SAS)
Event Hubs también proporciona **firmas de acceso compartido (SAS)** para el acceso delegado a recursos de Event Hubs para Kafka. La autorización del acceso mediante el mecanismo basado en tokens de OAuth 2.0 proporciona una mayor seguridad y facilidad de uso que SAS. Los roles integrados también pueden eliminar la necesidad de autorización basada en ACL, que el usuario debe mantener y administrar. Puede usar esta característica con los clientes de Kafka si especifica **SASL_SSL** para el protocolo y **PLAIN** para el mecanismo. 

```properties
bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";
```

> [!IMPORTANT]
> Reemplace `{YOUR.EVENTHUBS.CONNECTION.STRING}` por la cadena de conexión para el espacio de nombres de Event Hubs. Para obtener instrucciones sobre cómo obtener la cadena de conexión, consulte [Obtención de una cadena de conexión de Event Hubs](event-hubs-get-connection-string.md). A continuación se muestra un ejemplo de configuración: `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`

> [!NOTE]
> Al usar la autenticación de SAS con los clientes de Kafka, las conexiones establecidas no se desconectan cuando se vuelve a generar la clave SAS. 


#### <a name="samples"></a>Ejemplos 
Para leer un **tutorial** con instrucciones paso a paso para crear un centro de eventos y acceder a él mediante SAS u OAuth, consulte [Inicio rápido: Streaming de datos con Event Hubs mediante el protocolo de Kafka](event-hubs-quickstart-kafka-enabled-event-hubs.md).

Para obtener más **ejemplos** que muestran cómo usar OAuth con Event Hubs para Kafka, vea [ejemplos en GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth).

## <a name="other-event-hubs-features"></a>Otras características de Event Hubs 

La característica Event Hubs para Apache Kafka es uno de los tres protocolos disponibles simultáneamente en Azure Event Hubs, que complementa a HTTP y AMQP. Puede escribir con cualquiera de estos protocolos y leer con los otros, de modo que los productores de Apache Kafka actuales pueden seguir publicando mediante Apache Kafka mientras el lector puede beneficiarse de la integración nativa con la interfaz AMPQ de Event Hubs, como Azure Stream Analytics o Azure Functions. A la inversa, puede integrar fácilmente Azure Event Hubs en redes de enrutamiento AMQP como punto de conexión de destino y, además, leer los datos mediante integraciones de Apache Kafka.  

Además, otras características de Event Hubs, como la [Captura](event-hubs-capture-overview.md), que permite un almacenamiento de archivos a largo plazo extremadamente económico mediante Azure Blob Storage y Azure Data Lake Storage, y la [Recuperación geográfica ante desastres](event-hubs-geo-dr.md) también funcionan con la característica Event Hubs para Kafka.

## <a name="apache-kafka-feature-differences"></a>Diferencias entre las características de Apache Kafka 

El objetivo de Event Hubs para Apache Kafka es proporcionar acceso a las funcionalidades de Azure Event Hubs a las aplicaciones que están bloqueadas en la API de Apache Kafka y que, de lo contrario, tendrían que estar respaldadas por un clúster de Apache Kafka. 

Como se ha explicado [anteriormente](#is-apache-kafka-the-right-solution-for-your-workload), la flota de Azure Messaging proporciona una cobertura enriquecida y sólida para una gran cantidad de escenarios de mensajería y, aunque las siguientes características no se admiten actualmente a través de la compatibilidad con Event Hubs para la API de Apache Kafka, se indica dónde y cómo está disponible la funcionalidad deseada.

### <a name="transactions"></a>Transacciones

[Azure Service Bus](../service-bus-messaging/service-bus-transactions.md) ofrece una sólida compatibilidad con transacciones que permite recibir y resolver mensajes y sesiones al enviar mensajes salientes resultantes del procesamiento de mensajes a varias entidades de destino bajo la protección de coherencia de una transacción. El conjunto de características no solo permite un procesamiento puntual de cada mensaje en una secuencia, sino que también evita el riesgo de que otro consumidor vuelva a procesar involuntariamente los mismos mensajes, como sería el caso con Apache Kafka. Service Bus es el servicio recomendado para las cargas de trabajo de mensajes transaccionales.

### <a name="compression"></a>Compresión

La característica de [compresión](https://cwiki.apache.org/confluence/display/KAFKA/Compression) del lado cliente de Apache Kafka comprime un lote de varios mensajes en un único mensaje en el lado del productor y descomprime el lote en el lado del consumidor. El agente de Apache Kafka trata el lote como un mensaje especial.

Esta característica está esencialmente en conflicto con el modelo de múltiples protocolos de Azure Event Hubs, que permite que los mensajes, incluso los enviados por lotes, se puedan recuperar de forma individual desde el agente y a través de cualquier protocolo. 

La carga de cualquier evento de Event Hubs es un flujo de bytes y el contenido se puede comprimir con el algoritmo que desee. El formato de codificación de Apache Avro admite la compresión de forma nativa.

### <a name="log-compaction"></a>Compactación del registro

La compactación del registro de Apache Kafka es una característica que permite expulsar todos los registros salvo el último de cada clave de una partición, lo que convierte un tema de Apache Kafka en un almacén de pares clave-valor en el que el último valor agregado invalida el anterior. Esta característica no está implementada actualmente en Azure Event Hubs. El patrón de almacenamiento de pares clave-valor, incluso con actualizaciones frecuentes, es mucho más compatible con los servicios de base de datos, como [Azure Cosmos DB](../cosmos-db/introduction.md). Consulte el tema [Proyección de registros](event-hubs-federation-overview.md#log-projections) en la guía de federación de Event Hubs para obtener más detalles. 

### <a name="kafka-streams"></a>Kafka Streams

Kafka Streams es una biblioteca de cliente para análisis de flujos que forma parte del proyecto de código abierto de Apache Kafka, pero es independiente del agente de flujo de eventos de Apache Kafka. 

La razón más habitual por la que los clientes de Azure Event Hubs solicitan la compatibilidad con Kafka Streams es su interés en el producto "ksqlDB" de Confluent. "ksqlDB" es un proyecto de código compartido patentado [con licencia](https://github.com/confluentinc/ksql/blob/master/LICENSE), de modo que ningún proveedor que "ofrezca software como servicio, plataforma como servicio, infraestructura como servicio u otros servicios en línea similares que compitan con productos o servicios de Confluent" tengan permiso para usar u ofrecer compatibilidad con "ksqlDB". En la práctica, si usa ksqlDB, debe operar Kafka personalmente o bien utilizar las ofertas de la nube de Confluent. Los términos de licencia también pueden afectar a los clientes de Azure que ofrecen servicios para un propósito excluido por la licencia.

Kafka Streams, de forma independiente y sin ksqlDB, tiene menos funcionalidades que muchos marcos y servicios alternativos, la mayoría de los cuales incluyen interfaces SQL de transmisión integradas, y todos ellos se integran con Azure Event Hubs actualmente:

- [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md)
- [Azure Synapse Analytics (mediante la característica de captura de Event Hubs)](../event-grid/event-grid-event-hubs-integration.md)
- [Azure Databricks](/azure/databricks/scenarios/databricks-stream-from-eventhubs)
- [Apache Samza](https://samza.apache.org/learn/documentation/latest/connectors/eventhubs)
- [Apache Storm](event-hubs-storm-getstarted-receive.md)
- [Spark de Apache](event-hubs-kafka-spark-tutorial.md)
- [Apache Flink](event-hubs-kafka-flink-tutorial.md)
- [Akka Streams](event-hubs-kafka-akka-streams-tutorial.md)

Los servicios y marcos de trabajo enumerados generalmente pueden adquirir flujos de eventos y datos de referencia directamente de un conjunto diverso de orígenes mediante adaptadores. Kafka Streams solo puede adquirir datos de Apache Kafka y, por tanto, los proyectos de análisis están bloqueados en Apache Kafka. Para usar datos de otros orígenes, es necesario importar primero los datos en Apache Kafka con el marco de Kafka Connect.

Si debe usar el marco de Kafka Streams en Azure, [Apache Kafka en HDInsight](../hdinsight/kafka/apache-kafka-introduction.md) le proporcionará esa opción. Apache Kafka en HDInsight proporciona un control total sobre todos los aspectos de configuración de Apache Kafka, al tiempo que se integra por completo con distintos aspectos de la plataforma Azure, como la colocación del dominio de errores o actualizaciones, el aislamiento de red o la supervisión de la integración. 

## <a name="next-steps"></a>Pasos siguientes
En este artículo se proporciona una introducción a Event Hubs para Kafka. Para más información, consulte [Guía del desarrollador de Apache Kafka para Azure Event Hubs](apache-kafka-developer-guide.md).
