---
title: Comparación de los servicios de mensajería de Azure
description: 'Describe los tres servicios de mensajería de Azure: Azure Event Grid, Event Hubs y Service Bus. Recomienda qué servicio usar para escenarios diferentes.'
ms.topic: overview
ms.date: 07/01/2021
ms.openlocfilehash: f4980ca3c3741645af03874c5d75d9ef8da6084b
ms.sourcegitcommit: 6bd31ec35ac44d79debfe98a3ef32fb3522e3934
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2021
ms.locfileid: "113218716"
---
# <a name="choose-between-azure-messaging-services---event-grid-event-hubs-and-service-bus"></a>Elija entre los servicios de mensajería de Azure: Event Grid, Event Hubs y Service Bus

Azure ofrece tres servicios que le ayudan en la entrega de eventos o mensajes en una solución. Estos servicios son los siguientes:

- Azure Event Grid
- Azure Event Hubs
- Azure Service Bus

Aunque son similares, cada servicio se designa para escenarios determinados. Este artículo describe las diferencias entre estos servicios y le ayuda a entender cuál elegir para su aplicación. En muchos casos, los servicios de mensajería son complementarios y se pueden usar conjuntamente.

## <a name="event-vs-message-services"></a>Servicios de eventos frente a los de mensajería
Hay una diferencia importante entre los servicios que entregan un evento y los que entregan un mensaje.

### <a name="event"></a>Evento
Un evento es una notificación ligera de una condición o un cambio de estado. El publicador del evento no tiene ninguna expectativa sobre cómo se trata el evento. El consumidor del evento decide qué hacer con la notificación. Los eventos pueden ser unidades discretas o parte de una serie.

Los eventos discretos notifican un cambio de estado y se puede actuar sobre ellos. Para realizar el paso siguiente, el consumidor solo necesita saber que algo ha pasado. Los datos de evento tienen información acerca de lo que ha ocurrido, pero no tienen los datos que desencadenaron el evento. Por ejemplo, un evento notifica a los consumidores que se ha creado un archivo. Puede tener información general acerca del archivo, pero no contiene el propio archivo. Los eventos discretos funcionan muy bien con soluciones sin servidor que necesitan escalarse.

Los eventos de serie notifican una condición y son analizables. Los eventos están ordenados en el tiempo e interrelacionados. El consumidor necesita la serie de eventos ordenada para analizar lo que ha pasado.

### <a name="message"></a>Message
Un mensaje son datos sin procesar producidos por un servicio que se consumen o almacenan en otro lugar. El mensaje contiene los datos que desencadenó la canalización del mensaje. El publicador del mensaje tiene una expectativa sobre la forma en que el consumidor trata el mensaje. Existe un contrato entre ambas partes. Por ejemplo, el publicador envía un mensaje con los datos sin procesar, espera que el consumidor cree un archivo a partir de esos datos y envía una respuesta cuando el trabajo finaliza.


## <a name="azure-event-grid"></a>Azure Event Grid
Event Grid es un panel posterior de eventos que habilita la programación orientada a eventos y reactiva. Utiliza el modelo de publicación-suscripción. Los publicadores emiten eventos, pero no tienen expectativas de cómo se administran los eventos. Los suscriptores deciden qué eventos quieren administrar.

Event Grid está totalmente integrado con los servicios de Azure y puede integrarse con servicios de terceros. Simplifica el consumo de eventos y reduce los costos mediante la eliminación de la necesidad de un sondeo constante. Event Grid enruta de forma eficiente y fiable eventos desde recursos de Azure y que no son de Azure. Distribuye los eventos a los puntos de conexión del suscriptor registrado. El mensaje de evento tiene la información que necesita para reaccionar a los cambios en los servicios y aplicaciones. Event Grid no es una canalización de datos y no entrega el objeto real que se ha actualizado.

Tiene las características siguientes:

- Dinámicamente escalable
- Bajo costo
- Sin servidor
- Al menos una entrega de un evento

Para más información, consulte la [información general de Event Grid](overview.md).

## <a name="azure-event-hubs"></a>Azure Event Hubs
Azure Event Hubs es una plataforma de streaming de macrodatos y un servicio de ingesta de eventos. Puede recibir y procesar millones de eventos por segundo. Facilita la captura, la retención y la reproducción de los datos de streaming de eventos y telemetría. Los datos pueden proceder de distintos orígenes simultáneos. Event Hubs permite que los datos de telemetría y eventos estén disponibles para varios servicios de análisis e infraestructuras de procesamiento de streaming. Está disponible como flujos de datos o lotes de eventos agrupados. Este servicio proporciona una solución única que permite una recuperación de datos rápida para el procesamiento en tiempo real y una reproducción repetida de los datos sin procesar almacenados. Puede capturar los datos de streaming en un archivo para su procesamiento y análisis.

Tiene las características siguientes:

- Baja latencia.
- Puede recibir y procesar millones de eventos por segundo.
- Al menos una entrega de un evento

Para obtener más información, consulte [Información general sobre Event Hubs](../event-hubs/event-hubs-about.md).

## <a name="azure-service-bus"></a>Azure Service Bus
Service Bus es un agente de mensajes empresarial totalmente administrado que incluye colas de mensajes y temas que se pueden publicar y a los que es posible suscribirse. El servicio está dirigido a aplicaciones empresariales tradicionales que requieren transacciones, ordenación, detección de duplicados y coherencia inmediata. Service Bus permite que las aplicaciones en la nube nativas proporcionen una administración de transición de estado de confianza para los procesos empresariales. Cuando se administran mensajes de gran valor que no se pueden perder o duplicarse, use Azure Service Bus. Este servicio también facilita la comunicación de alta seguridad en soluciones híbridas en la nube y pueden conectar sistemas locales existentes a soluciones en la nube.

Service Bus es un sistema de mensajería asincrónica. Almacena los mensajes en un "agente" (por ejemplo, una cola) hasta que la parte consumidora esté preparada para recibirlos. Tiene las características siguientes:

- Entrega de mensajes asincrónicos de confianza (mensajería empresarial como servicio) que requiere un sondeo.
- Características de mensajería avanzada, como primero en entrar, primero en salir (FIFO), procesamiento por lotes o sesiones, transacciones, colas de mensajes fallidos, control temporal, enrutamiento y filtrado, así como detección de duplicados
- Al menos una entrega de un mensaje
- Entrega ordenada opcional de mensajes

Para más información, consulte la [información general de Service Bus](../service-bus-messaging/service-bus-messaging-overview.md).

## <a name="comparison-of-services"></a>Comparación de servicios

| Servicio | Propósito | Tipo | Cuándo se usa |
| ------- | ------- | ---- | ----------- |
| Event Grid | Programación reactiva | Distribución de eventos (discretos) | Reacción ante los cambios de estado |
| Event Hubs | Canalización de macrodatos | Streaming de eventos (serie) | Streaming de datos distribuidos y telemetría |
| Azure Service Bus | Mensajería empresarial de gran valor | Message | Procesamiento de pedidos y transacciones financieras |

## <a name="use-the-services-together"></a>Uso conjunto de servicios
En algunos casos, puede utilizar servicios en paralelo para cumplir con distintos roles. Por ejemplo, un sitio de comercio electrónico puede utilizar Service Bus para procesar el pedido, Event Hubs para capturar la telemetría del sitio y Event Grid para responder a eventos, como un artículo enviado.

En otros casos, puede vincular estas soluciones para formar una canalización de datos y eventos. Use Event Grid para responder a los eventos en los demás servicios. Para obtener un ejemplo sobre el uso de Event Grid con Event Hubs para migrar datos a un almacén de datos, vea [Transmisión de macrodatos a un almacén de datos](event-grid-event-hubs-integration.md). En la siguiente imagen se muestra el flujo de trabajo para el streaming de datos.

:::image type="content" source="./media/compare-messaging-services/overview.png" alt-text="Imagen que muestra cómo Event Hubs, Service Bus y Event Grid se pueden conectar entre sí":::.

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes: 
- [Opciones de mensajería asincrónica en Azure](/azure/architecture/guide/technology-choices/messaging)
- [Events, Data Points, and Messages - Choosing the right Azure messaging service for your data](https://azure.microsoft.com/blog/events-data-points-and-messages-choosing-the-right-azure-messaging-service-for-your-data/) (Eventos, puntos de datos y mensajes - Elección del servicio de mensajería de Azure adecuado para los datos).

