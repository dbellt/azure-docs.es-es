---
title: Archivo de inclusión
description: archivo de inclusión
services: service-bus-messaging
author: clemensv
ms.service: service-bus-messaging
ms.topic: include
ms.date: 04/08/2021
ms.author: clemensv
ms.custom: include file
ms.openlocfilehash: 08fccf366321b075542f36b86c9bf22d5d877167
ms.sourcegitcommit: 5163ebd8257281e7e724c072f169d4165441c326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/21/2021
ms.locfileid: "112415687"
---
La opción de protocolo AMQP sobre WebSockets se ejecuta a través del puerto TCP 443 como la API HTTP o de REST, pero, de lo contrario, es funcionalmente idéntica a AMQP sin modificar. Esta opción tiene una mayor latencia de conexión inicial debido a los intercambios de protocolo de enlace adicionales y una sobrecarga ligeramente mayor como compensación para compartir el puerto HTTPS. Si se selecciona este modo, el puerto TCP 443 es suficiente para la comunicación. Las opciones siguientes permiten seleccionar el modo WebSockets AMQP. 

| Lenguaje | Opción   |
| -------- | ----- |
| .NET (Azure.Messaging.ServiceBus)    | Cree [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient.-ctor) mediante un constructor que adopte [ServiceBusClientOptions](/dotnet/api/azure.messaging.servicebus.servicebusclientoptions) como parámetro. Establezca [ServiceBusClientOptions.TransportType](/dotnet/api/azure.messaging.servicebus.servicebusclientoptions.transporttype) en [ServiceBusTransportType.AmqpWebSockets](/dotnet/api/azure.messaging.servicebus.servicebustransporttype). |
| .NET (Microsoft.Azure.ServiceBus)    | Al crear objetos de cliente, use constructores que adoptan [TransportType](/dotnet/api/microsoft.azure.servicebus.transporttype), [ServiceBusConnection](/dotnet/api/microsoft.azure.servicebus.servicebusconnection) o [ServiceBusConnectionStringBuilder](/dotnet/api/microsoft.azure.servicebus.servicebusconnectionstringbuilder) como parámetros. <p>Para la construcción que adopta `transportType` como parámetro, establezca el parámetro en [TransportType.AmqpWebSockets](/dotnet/api/microsoft.azure.servicebus.transporttype).</p> <p>Para el constructor que adopta `ServiceBusConnection` como parámetro, establezca [ServiceBusConnection.TransportType](/dotnet/api/microsoft.azure.servicebus.servicebusconnection.transporttype) en [TransportType.AmqpWebSockets](/dotnet/api/microsoft.azure.servicebus.transporttype).</p> <p>Si utiliza `ServiceBusConnectionStringBuilder`, use constructores que le den una opción para especificar `transportType`.</p> |
| Java (com.azure.messaging.servicebus)     | Al crear clientes, establezca [ServiceBusClientBuilder.transportType](/java/api/com.azure.messaging.servicebus.servicebusclientbuilder.transporttype) en [AmqpTransportType.AMQP.AMQP_WEB_SOCKETS](/java/api/com.azure.core.amqp.amqptransporttype). |
| Java (com.microsoft.azure.servicebus)    | Al crear clientes, establezca `transportType` de [com.microsoft.azure.servicebus.ClientSettings](/java/api/com.microsoft.azure.servicebus.clientsettings.clientsettings#com_microsoft_azure_servicebus_ClientSettings_ClientSettings_com_microsoft_azure_servicebus_security_TokenProvider_com_microsoft_azure_servicebus_primitives_RetryPolicy_java_time_Duration_com_microsoft_azure_servicebus_primitives_TransportType_) en [com.microsoft.azure.servicebus.primitives.TransportType.AMQP_WEB_SOCKETS](/java/api/com.microsoft.azure.servicebus.primitives.transporttype). |
| JavaScript  | Al crear objetos de cliente de Service Bus, use la propiedad `webSocketOptions` en [ServiceBusClientOptions](/javascript/api/@azure/service-bus/servicebusclientoptions). |
| Python | Al crear clientes de Service Bus, establezca [ServiceBusClient.transport_type](/python/api/azure-servicebus/azure.servicebus.servicebusclient) en [TransportType.AmqpOverWebSocket](/python/api/azure-servicebus/azure.servicebus.transporttype). |

