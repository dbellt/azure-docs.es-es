---
title: archivo de inclusión
description: archivo de inclusión
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: include
ms.date: 01/21/2021
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 2970a7271761082a358b50c069da82ed785df8b0
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107990478"
---
### <a name="what-ports-do-i-need-to-open-on-the-firewall"></a>¿Qué puertos es necesario abrir en el firewall? 
Puede usar los siguientes protocolos con Azure Event Hubs para enviar y recibir eventos:

- Advanced Message Queuing Protocol 1.0 (AMQP)
- Protocolo de transferencia de hipertexto 1.1 con TLS (HTTPS)
- Apache Kafka

Consulte en la siguiente tabla los puertos de salida que se deben abrir para usar estos protocolos para comunicarse con Azure Event Hubs. 

| Protocolo | Puertos | Detalles | 
| -------- | ----- | ------- | 
| AMQP | 5671 y 5672 | Consulte la [guía del protocolo AMQP](../articles/service-bus-messaging/service-bus-amqp-protocol-guide.md) | 
| HTTPS | 443 | Este puerto se usa para la API HTTP/REST y para AMQP a través de WebSockets. |
| Kafka | 9093 | Consulte [Uso de Azure Event Hubs desde aplicaciones de Apache Kafka](../articles/event-hubs/event-hubs-for-kafka-ecosystem-overview.md)

El puerto HTTPS también es necesario para la comunicación saliente cuando se usa AMQP a través del puerto 5671, ya que varias operaciones de administración realizadas por los SDK de cliente y la adquisición de tokens desde Azure Active Directory (cuando se usa) se ejecutan a través de HTTPS. 

Los SDK de Azure oficiales suelen usar el protocolo AMQP para enviar y recibir eventos de Event Hubs. La opción de protocolo AMQP sobre WebSockets se ejecuta a través del puerto TCP 443 como la API HTTP, pero, de lo contrario, es funcionalmente idéntica a AMQP sin modificar. Esta opción tiene una mayor latencia de conexión inicial debido a los intercambios de protocolo de enlace adicionales y una sobrecarga ligeramente mayor como compensación para compartir el puerto HTTPS. Si se selecciona este modo, el puerto TCP 443 es suficiente para la comunicación. Las opciones siguientes permiten seleccionar el modo WebSockets AMQP o AMQP:

| Lenguaje | Opción   |
| -------- | ----- |
| .NET     | Propiedad [EventHubConnectionOptions.TransportType](/dotnet/api/azure.messaging.eventhubs.eventhubconnectionoptions.transporttype) con [EventHubsTransportType.AmqpTcp](/dotnet/api/azure.messaging.eventhubs.eventhubstransporttype) o [EventHubsTransportType.AmqpWebSockets](/dotnet/api/azure.messaging.eventhubs.eventhubstransporttype) |
| Java     | [com.microsoft.azure.eventhubs.EventProcessorClientBuilder.transporttype](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/src/main/java/com/azure/messaging/eventhubs/EventProcessorClientBuilder.java) con [AmqpTransportType.AMQP](/java/api/com.azure.core.amqp.amqptransporttype) o [AmqpTransportType.AMQP_WEB_SOCKETS](/java/api/com.azure.core.amqp.amqptransporttype) |
| Nodo  | [EventHubConsumerClientOptions](/javascript/api/@azure/event-hubs/eventhubconsumerclientoptions) tiene una propiedad `webSocketOptions`. |
| Python | [EventHubConsumerClient.transport_type](/python/api/azure-eventhub/azure.eventhub.eventhubconsumerclient) con [TransportType.Amqp](/python/api/azure-eventhub/azure.eventhub.transporttype) o [TransportType.AmqpOverWebSocket](/python/api/azure-eventhub/azure.eventhub.transporttype) |

### <a name="what-ip-addresses-do-i-need-to-allow"></a>¿Qué direcciones IP tengo que permitir?
Cuando trabaja con Azure, en ocasiones tiene que permitir intervalos de direcciones IP específicos o direcciones URL en el firewall o proxy corporativo para tener acceso a todos los servicios de Azure que está usando o intentando usar. Compruebe que se permite el tráfico en las direcciones IP utilizadas por Event Hubs. En el caso de las direcciones IP usadas por Azure Event Hubs, vea [Intervalos de direcciones IP y etiquetas de servicio de Azure: nube pública](https://www.microsoft.com/download/details.aspx?id=56519).

Además, compruebe que se permite la dirección IP del espacio de nombres. Para buscar las direcciones IP correctas para permitirlas para las conexiones, siga estos pasos:

1. Ejecute el siguiente comando desde el símbolo del sistema: 

    ```
    nslookup <YourNamespaceName>.servicebus.windows.net
    ```
2. Anote la dirección IP devuelta en `Non-authoritative answer`. 

Si usa la **redundancia de zona** para el espacio de nombres, deberá realizar algunos pasos adicionales: 

1. En primer lugar, ejecute nslookup en el espacio de nombres.

    ```
    nslookup <yournamespace>.servicebus.windows.net
    ```
2. Anote el nombre de la sección **respuesta no autoritativa**, que se encuentra en uno de los siguientes formatos: 

    ```
    <name>-s1.cloudapp.net
    <name>-s2.cloudapp.net
    <name>-s3.cloudapp.net
    ```
3. Ejecute nslookup para cada uno con los sufijos s1, s2 y s3 para obtener las direcciones IP de las tres instancias que se ejecutan en tres zonas de disponibilidad. 

    > [!NOTE]
    > La dirección IP devuelta por el comando `nslookup` no es una dirección IP estática. Sin embargo, permanece constante hasta que la implementación subyacente se elimine o se mueva a otro clúster.

### <a name="what-client-ips-are-sending-events-to-or-receiving-events-from-my-namespace"></a>¿Qué direcciones IP de cliente envían o reciben eventos desde mi espacio de nombres?
En primer lugar, habilite el [filtrado de IP](../articles/event-hubs/event-hubs-ip-filtering.md) en el espacio de nombres. 

A continuación, habilite los registros de diagnóstico para [eventos de conexión de red virtual de Event Hubs](../articles/event-hubs/event-hubs-diagnostic-logs.md#event-hubs-virtual-network-connection-event-schema) siguiendo las instrucciones indicadas en [Habilitar registros de diagnóstico](../articles/event-hubs/event-hubs-diagnostic-logs.md#enable-diagnostic-logs). Verá la dirección IP para la que se deniega la conexión.

```json
{
    "SubscriptionId": "0000000-0000-0000-0000-000000000000",
    "NamespaceName": "namespace-name",
    "IPAddress": "1.2.3.4",
    "Action": "Deny Connection",
    "Reason": "IPAddress doesn't belong to a subnet with Service Endpoint enabled.",
    "Count": "65",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name",
    "Category": "EventHubVNetConnectionEvent"
}
```

> [!IMPORTANT]
> Los registros de red virtual solo se generan si el espacio de nombres permite el acceso desde **direcciones IP específicas** (reglas de filtro de IP). Si no desea restringir el acceso al espacio de nombres mediante estas características y desea obtener registros de red virtual para realizar el seguimiento de las direcciones IP de los clientes que se conectan al espacio de nombres de Event Hubs, puede usar la siguiente alternativa: Habilite el filtrado de IP y agregue el intervalo IPv4 direccionable total (1.0.0.0/1 - 255.0.0.0/1). Event Hubs no admite los intervalos de direcciones IPv6. 

> [!NOTE]
> Actualmente no es posible determinar la IP de origen de un mensaje o evento individual. 
